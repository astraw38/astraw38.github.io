---
layout: post
title: Customizing Pytest Collection
published: false
---

# Customing Pytest Collection

## Examples



### Using pytest_generate_tests

The easiest method to generate tests from existing data is to use 

```py
import pytest
import json
import os

TEST_DATA_LOC = "testdata"


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


class TestJson(object):
    def test_individual_json(self, json_file):
        json.load(json_file)

```

### Manual collection

We can influence how tests are created as well as their ordering by overriding internal pytest objects 
and implementing the ``collect()`` function as below. 

This is useful if you'd prefer have external names of test functions or influence ordering of tests within a test
class.

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
