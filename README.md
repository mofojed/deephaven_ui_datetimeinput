# deephaven_ui_datetimeinput

This is a Python plugin for Deephaven generated from a [deephaven-plugin](https://github.com/deephaven/deephaven-plugins) template.

Specifically, this plugin is a bidirectional widget plugin, which can send and receive messages on both the client and server.
The plugin works out of the box, demonstrates basic plugin structure, and can be used as a starting point for building more complex plugins.

## Plugin Structure

The `src` directory contains the Python and JavaScript code for the plugin.
Within the `src` directory, the deephaven_ui_datetimeinput directory contains the Python code, and the `js` directory contains the JavaScript code.

The Python files have the following structure:
`deephaven_ui_datetimeinput_object.py` defines a simple Python class that can send messages to the client.
`deephaven_ui_datetimeinput_type.py` defines the Python type for the plugin (which is used for registration) and a simple message stream.
`js_plugin.py` defines the Python class that will be used to setup the JavaScript side of the plugin.
`register.py` registers the plugin with Deephaven.

The JavaScript files have the following structure:
`DeephavenUiDateTimeInputPlugin.ts` registers the plugin with Deephaven.
`DeephavenUiDateTimeInputView.tsx` defines the plugin panel and message handling.

Additionally, the `test` directory contains Python tests for the plugin. This demonstrates how the embedded Deephaven server can be used in tests.
It's recommended to use `tox` to run the tests, and the `tox.ini` file is included in the project.

## Building the Plugin

To build the plugin, you will need `npm` and `python` installed, as well as the `build` package for Python.
`nvm` is also strongly recommended, and an `.nvmrc` file is included in the project.
The python venv can be created and the recommended packages installed with the following commands:

```sh
cd deephaven_ui_datetimeinput
python -m venv .venv
source .venv/bin/activate
pip install --upgrade -r requirements.txt
```

Build the JavaScript plugin from the `src/js` directory:

```sh
cd src/js
nvm install
npm install
npm run build
```

Then, build the Python plugin from the top-level directory:

```sh
cd ../..
python -m build --wheel
```

The built wheel file will be located in the `dist` directory.

## Installing the Plugin

The plugin can be installed into a Deephaven instance with `pip install <wheel file>`.
The wheel file is stored in the `dist` directory after building the plugin.
Exactly how this is done will depend on how you are running Deephaven.
If using the venv created above, the plugin and server can be created with the following commands:

```sh
pip install deephaven-server
pip install dist/deephaven_ui_datetimeinput-0.0.1.dev0-py3-none-any.whl --force-reinstall
deephaven server
```

See the [plug-in documentation](https://deephaven.io/core/docs/how-to-guides/use-plugins/) for more information.

## Using the Plugin

Once the Deephaven server is running, the plugin should be available to use.

```python
from deephaven_ui_datetimeinput import DateTimeInput

dti = DateTimeInput(on_change=print)
```

A panel should appear. If you make changes in the input, they should get printed to your console.

For a more complete example where you filter a table using the date inputted:

```python
import datetime
from deephaven import time_table, ui
from deephaven_ui_datetimeinput import DateTimeInput

# Create some dates and a time table that goes back in time one year
now = datetime.datetime.now()
one_year_earlier = now - datetime.timedelta(days=365)
tt = time_table("PT1s", start_time=one_year_earlier)

# Create a component that uses the `DateTimeInput` to filter a
@ui.component
def ui_time_filter_table(source):
  date, set_date = ui.use_state('2024-05-21T12:00:00.000000000 America/Toronto')

  return [
    DateTimeInput(on_change=set_date, default_value=date),
    source.where(f"Timestamp > '{date}'")
  ]

tft = ui_time_filter_table(tt)
```

## Distributing the Plugin

To distribute the plugin, you can upload the wheel file to a package repository, such as [PyPI](https://pypi.org/).
The version of the plugin can be updated in the `setup.cfg` file.

There is a separate instance of PyPI for testing purposes.
Start by creating an account at [TestPyPI](https://test.pypi.org/account/register/).
Then, get an API token from [account management](https://test.pypi.org/manage/account/#api-tokens), setting the “Scope” to “Entire account”.

To upload to the test instance, use the following commands:

```sh
python -m pip install --upgrade twine
python -m twine upload --repository testpypi dist/*
```

Now, you can install the plugin from the test instance. The extra index is needed to find dependencies:

```sh
pip install --index-url https://test.pypi.org/simple/ --extra-index-url https://pypi.org/simple/ deephaven_ui_datetimeinput
```

For a production release, create an account at [PyPI](https://pypi.org/account/register/).
Then, get an API token from [account management](https://pypi.org/manage/account/#api-tokens), setting the “Scope” to “Entire account”.

To upload to the production instance, use the following commands.
Note that `--repository` is the production instance by default, so it can be omitted:

```sh
python -m pip install --upgrade twine
python -m twine upload dist/*
```

Now, you can install the plugin from the production instance:

```sh
pip install deephaven_ui_datetimeinput
```

See the [Python packaging documentation](https://packaging.python.org/en/latest/tutorials/packaging-projects/#uploading-the-distribution-archives) for more information.
