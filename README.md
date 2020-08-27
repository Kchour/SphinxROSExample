# SphinxROSExample

Sphinx is an open source document generation tool for python projects. You can create really elegant html or even pdf documents for your code. Though Sphinx is a great tool, it can be overwhelming to set up. 

This repo will help you setup Sphinx for your Python-based ROS packages. Instructions adapted from ROS wiki pages (see additional resources)

# Requirements
- Python 3. However, it is still possible to document python2 code with sphinx-python3 
- Sphinx
- m2r2
- sphinx-rtd-theme
- rosdoc_lite 

Install python requirements by using 
```
python3 -m pip install -r requirements.txt
```

Also install rosdoc_lite using
```
sudo apt install ros-melodic-rosdoc-lite
```

Note: Code must be documented using `docstrings`. I have opted for [Googlestyle docstrings](https://sphinxcontrib-napoleon.readthedocs.io/en/latest/example_google.html)

# Instructions
- cd to the top level of your ROS package
- Create an empty folder called `doc` and go inside it

## ROS Preliminaries
- Enable rosdoc-lite support of sphinx, by adding the following to your `package.xml` manifest file:

```
<export>
  <rosdoc config="rosdoc.yaml" />
</export>
```
- Also, in the top-level directory of your ROS package, add `rosdoc.yaml` with the following:

```
- builder: sphinx           # specify document generator. e.g) doxygen, epidoc, sphinx
  sphinx_root_dir: doc      # document directory
```

## Sphinx Quickstart

For any of the options, pressing 'enter' will utilize the default values.

- Run sphinx's quickstart command: `$ sphinx-quickstart`
- Say `No` to `Separate source and build directories (y/n) [n]: `
- Name your project
- Give the author/maintainer name
- Give a version number (I try to follow semantic versioning X.Y.Z). However, this can be automated with a few steps later
- Choose your desired language

## Edit `conf.py`
Remember that Sphinx is not totally automatic, so we need to tune a few settings for it to work. Using your favorite editor, open `doc/conf.py`.
- Tell sphinx where your package is located using [catkin](http://wiki.ros.org/Sphinx), by adding the following:
```
 import os
 import catkin_pkg.package
 catkin_dir = os.path.dirname(os.path.dirname(os.path.abspath(__file__))) 
 catkin_package = catkin_pkg.package.parse_package(os.path.join(catkin_dir, catkin_pkg.package.PACKAGE_MANIFEST_FILENAME))
```
- If you want to pull version numbers from your manifest, add the following too:
```
 version = catkin_package.version
 release = catkin_package.version
```
- Add a few extensions [extensions](https://www.sphinx-doc.org/en/master/usage/extensions/index.html) for extra Sphinx functionality
```
extensions = [
        'm2r2',
        'sphinx.ext.autodoc',
        'sphinx.ext.napoleon',
        'sphinx.ext.viewcode',
        'sphinx.ext.todo',
        'sphinx.ext.autosummary',
]
autosummary_generate = True			#Added this too
source_suffix = ['.rst', '.md']		#Added this too
```

- Finally, you can change the html theme (user interface basically). Lots of options to choose from, but let's use following:
```
html_theme = `sphinx_rtd_theme`
```

- Save and close!

## Edit `index.rst`

We will also need to edit `index.rst`. Do recall that .rst extension refers to [restructuredText](https://docutils.sourceforge.io/rst.html), which is an alternative to markdown style readme's you may be used to. 

- Change it so that `index.rst` looks like the following:

```

.. toctree::

	readme_link

.. autosummary::
	:toctree: _autosummary
	:caption: API Reference
	:template: custom-module-template.rst
	:recursive:

	example_package


```

- `readme_link` above will refer to external markdown files of your own. If you don't have one, then you can safely remove this. More on this in the next section
- Make sure to replace `example_package` with your ROS package name

## Include *.md Files

You'll notice that we have a 'README.md'. Can sphinx use them? The answer is yes, but you have to add a few files.

- Assuming you have completed the previous section, add a `readme_link.rst` file to your `doc` folder containing the following:
```
.. mdinclude:: ../README.md
```
This will help sphinx locate your `.md` files and convert them to `.rst` format internally. Note: Do the same thing for additional markdown files you may have.


## Add Template Files
The latest version of sphinx (at the time of this writing) has the ability to recursively scan your package and generate nice API documentation, a feature called "autosummary". To use the autosummary recursive option, we need to provide a format for sphinx (courtesy of [JamesALeedham](https://github.com/JamesALeedham/Sphinx-Autosummary-Recursion)).

- Add `custom-class-template.rst` to `doc/_template/` with the following:

```
{{ fullname | escape | underline}}

.. currentmodule:: {{ module }}

.. autoclass:: {{ objname }}
   :members:
   :show-inheritance:
   :inherited-members:
   :special-members: __call__, __add__, __mul__

   {% block methods %}
   {% if methods %}
   .. rubric:: {{ _('Methods') }}

   .. autosummary::
      :nosignatures:
   {% for item in methods %}
      {%- if not item.startswith('_') %}
      ~{{ name }}.{{ item }}
      {%- endif -%}
   {%- endfor %}
   {% endif %}
   {% endblock %}

   {% block attributes %}
   {% if attributes %}
   .. rubric:: {{ _('Attributes') }}

   .. autosummary::
   {% for item in attributes %}
      ~{{ name }}.{{ item }}
   {%- endfor %}
   {% endif %}
   {% endblock %}

```
- Also add `custom-module-template.rst` to the same directory with:

```
{{ fullname | escape | underline}}

.. automodule:: {{ fullname }}

   {% block attributes %}
   {% if attributes %}
   .. rubric:: Module attributes

   .. autosummary::
      :toctree:
   {% for item in attributes %}
      {{ item }}
   {%- endfor %}
   {% endif %}
   {% endblock %}

   {% block functions %}
   {% if functions %}
   .. rubric:: {{ _('Functions') }}

   .. autosummary::
      :toctree:
      :nosignatures:
   {% for item in functions %}
      {{ item }}
   {%- endfor %}
   {% endif %}
   {% endblock %}

   {% block classes %}
   {% if classes %}
   .. rubric:: {{ _('Classes') }}

   .. autosummary::
      :toctree:
      :template: custom-class-template.rst
      :nosignatures:
   {% for item in classes %}
      {{ item }}
   {%- endfor %}
   {% endif %}
   {% endblock %}

   {% block exceptions %}
   {% if exceptions %}
   .. rubric:: {{ _('Exceptions') }}

   .. autosummary::
      :toctree:
   {% for item in exceptions %}
      {{ item }}
   {%- endfor %}
   {% endif %}
   {% endblock %}

{% block modules %}
{% if modules %}
.. autosummary::
   :toctree:
   :template: custom-module-template.rst
   :recursive:
{% for item in modules %}
   {{ item }}
{%- endfor %}
{% endif %}
{% endblock %}

```

## Build Docs
- Go to the top level of your package and run `$ rosdoc_lite .`

## View Docs

If all is well, the doc will be generated at `doc/html/index.html`. Open with your browser to view. Done!

## Additional Resources
- [My Sphinx tutorial for typical Python packages, not ROS](https://github.com/Kchour/SphinxExample)
- [ROS wiki on Sphinx](http://wiki.ros.org/Sphinx)
- [ROS wiki on rosdoc-lite](http://wiki.ros.org/rosdoc_lite)
