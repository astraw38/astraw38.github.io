---
layout: post
title: Customizing Pytest Collection
published: false
---

# Customing Pytest Collection

## Examples



### Using pytest_generate_tests

The easiest method to 

```py
import pytest
import json
import os
from os.path import splitext

TEST_DATA_LOC = "testdata"

def pytest_generate_tests(metafunc):
    if "json_file" in metafunc.fixturenames:
        test_json_files = [x for x in os.listdir(TEST_DATA_LOC)]
        metafunc.parametrize("json_file", test_json_files, indirect=True, scope='function')
    
@pytest.fixture(scope='function')
def json_file(request):
    with open(request.param) as myf:
        yield myf
    
class TestJson(object):
    def test_individual_json(self, json_file):
        json.load(json_file)
```
