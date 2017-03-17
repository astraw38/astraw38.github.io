---
layout: post
title: Data Driven Pytest
published: false
---

# Data Driven Pytest

As a tester, I have found that there is a huge amount of testing that is done using the exact same process, only varying data inputs and validating the output. 

The most basic method in pytest of running various data through a single test process is to use *parametrization*. You can parametrize fixtures or test functions, but you still need to generate the data yourself and update it as your product and tests change. One amazing tool to automatically generate a wide variety of data is [Hypothesis](https://hypothesis.readthedocs.io/en/latest/), but it has its limitations.

At my work, I had a large amount of pre-existing test data that I wanted to integrate into our testing framework. I found that our test data did not integrate well with the existing methods of generating tests (raw parametrization or ``pytest_generate_tests``), leading me on a journey far into the internals of pytest to figure out how to influence test collection methods and order. 


In the examples below I show some various ways to read in external data as tests in Pytest, ranging from the easist to the more difficult (but most flexible). 


## Examples

### Raw parametrization

This is useful if you have your data accessible at module load time, but can lead to 
all of your tests failing to run with a collection failure. 

Note that if your fixture is in a conftest, then the ``os.listdir`` will be **executed** long before 
your commandline options are parsed. If you need to delay the parametrization of the tests then use
``pytest_generate_tests`` (example below). 

```py
import pytest
import os
import json

TEST_DATA_LOC = "..{}testdata".format(os.path.sep)


@pytest.fixture(scope='function', params=os.listdir(TEST_DATA_LOC))
def json_data(request):
    with open(os.path.join(TEST_DATA_LOC, request.param)) as jdata:
         yield jdata


def test_json_load(json_data):
    json.load(json_data)

```

Output:
```
test_fixture.py::test_json_load[array.json] PASSED
test_fixture.py::test_json_load[glossary.json] PASSED
test_fixture.py::test_json_load[nested.json] PASSED
```

### Using pytest_generate_tests

This is effectively the same as above, but parametrization of the fixture is delayed until the ``pytest_generate_tests`` hook
is called. 

``if 'json_file' in metafunc.fixturenames`` is necessary if you have your ``pytest_generate_tests`` hook in a conftest that is above
multiple test files that don't all use that fixture. 

```py
import pytest
import json
import os

TEST_DATA_LOC = "..{}testdata".format(os.path.sep)


def pytest_generate_tests(metafunc):
    if "json_file" in metafunc.fixturenames:
        test_json_files = [x for x in os.listdir(TEST_DATA_LOC)]
        metafunc.parametrize("json_file", test_json_files,
                             ids=[x.replace(".", "_") for x in test_json_files],
                             indirect=True, scope='function')


@pytest.fixture(scope='function')
def json_file(request):
    with open(os.path.join(TEST_DATA_LOC, request.param)) as myf:
        yield myf


def test_json_load(json_file):
    json.load(json_file)

```

Output:
```
test_generate_tests.py::test_json_load[array_json] PASSED
test_generate_tests.py::test_json_load[glossary_json] PASSED
test_generate_tests.py::test_json_load[nested_json] PASSED
```

### Manual collection

We can influence how tests are created as well as their ordering by overriding internal pytest objects 
and implementing the ``collect()`` function as below. 

This is useful if you'd prefer have external names of test functions (instead of a single testname + the params) or need to influence ordering of tests within a test class.


``conftest.py``
```py
import inspect
import glob
import os

import pytest
from _pytest.python import Class, Instance

from manual_collection.mixin import JsonTesterMixin


@pytest.hookimpl(hookwrapper=True)
def pytest_pycollect_makeitem(collector, name, obj):
    outcome = yield
    if name.startswith("Test") and inspect.isclass(obj) and issubclass(obj, JsonTesterMixin):
        outcome.force_result(JsonCollectorClass(name, collector))


class JsonCollectorClass(Class):
    def collect(self):
        test_class = self.obj

        dataloc = getattr(test_class, "TESTDATA", "*")
        test_paths = glob.glob(dataloc)
        for item in test_paths:
            test_name = "test_{}".format(os.path.basename(item).replace(".", "_"))
            setattr(test_class, test_name, test_class.build_test_method(item))

        return [JsonCollectorInstance("()", self)]


class JsonCollectorInstance(Instance):
    def collect(self):
        # Delegate the collection of test methods to _pytest.PyCollector
        items = super(JsonCollectorInstance, self).collect()
        # But
        return sorted(items, key=lambda x: x.name, reverse=True)

```

``mixin.py``
```py
import json


class JsonTesterMixin(object):
    @classmethod
    def build_test_method(cls, data_path):
        def test_base(self):
            with open(data_path) as data:
                json.load(data)

        return test_base
```

``test_collection.py``
```py
import os

from manual_collection.mixin import JsonTesterMixin


class TestJson(JsonTesterMixin):
    TESTDATA = "..{0}testdata{0}*".format(os.path.sep)

```

Output:

```
test_collection.py::TestJson::test_nested_json <- mixin.py PASSED
test_collection.py::TestJson::test_glossary_json <- mixin.py PASSED
test_collection.py::TestJson::test_array_json <- mixin.py PASSED
```
