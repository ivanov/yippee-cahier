#####################
Design with use-cases
#####################

We like to write tutorials and other documents in restructured text, and
particularly with sphinx.

This allows us to use - say - vim - for editing text and code.

The IPython notebook is pleasantly interactive in showing us rendering of LaTeX,
and output of code execution, but the web interface feels heavy and
uncomfortable when writing anything longer than a few pages.

*********
Use-cases
*********

* I write a short snippet of code in a RST file in vim.  From vim I cause the
  code to be run, and the results are shown in the web browser, including
  generated plots.  I can cause the printed results from the notebook to be
  output into my current text buffer in vim;
* When building the final page from the RST document, the rendered plots from
  the web browser will appear as images in the rendered page;
* From vim I can cause some current part of text to be converted to markdown and
  rendered in the web browser.  This allows me to review LaTeX rendered with
  MathJax or similar;
* As I edit, I may want to keep a selection of rendered results current in the
  web browswer to remind me what the rendered output will look like;
* I will often want to check the results of the code in the RST file still hold
  after my edits.  I can run some command from vim to rerun all the code
  snippets in the vim buffer in a fresh execution environment and replace the
  rendered results in the web browser.

*********************
Implementation sketch
*********************

We edit text on the (say) left of the screen, in vim, and review output on the
(say) right of the screen, in some version of the IPython notebook.

This is *edit left, look right*.

We stored the results that are the basis for the display on the right, as an
IPython notebook, in notebook JSON format.  We use this notebook as store for
the results, and we will fetch the assets (outputs) from this store when
creating the built version of the RST page.

We will call the stored JSON notebook ``scratch.ipynb``, although its actual
name will probably be some version of the input RST document.

Math / markdown rendering
=========================

This can either be:

* The current selected block of text; or
* (No text selected) all text above and below the cursor up to (down to) any
  directive apart from a ``..math`` directive, or a heading.

We convert this text to markdown according to the following rules:

* We discard any directives or inline text roles that are not ``..math::`` or
  ``:math:``;
* We replace inline or directive ``math`` with text between dollars or text
  between double dollar markers respectively;

The resulting cell appears last in the rendered notebook.

Code execution and rendering
============================

Code for execution is in custom code directives::

    .. icode::

        a = 10
        print(a)

On running some vim function such as ``:ExecICode``, vim does the following:

* finds the code surrounding the cursor position and stores as
  ``code-block``;
* creates a UUID for this block if not already specified in the directive
  options;
* writes the UUID into the text of the directive, to form something like::

    .. icode::
        :uuid: 12ab23cd45ef

        a = 10
        print(a)

* executes this code in the ipython notebook to the right.  The execution code
  first checks the notebook to see if there is a cell with metadata containing
  this UUID value.  If so, the code input and output text go into the
  pre-existing cell in the notebook.  Otherwise the notebook creates and fills a
  new cell.  The UUID appears in the metadata for the cell in the
  ``scratch.ipynb`` notebook;
* I (the user) can execute a command like ``:AlwaysShow`` in the text buffer,
  such that the result of running this cell, whether refreshing and old cell or
  creating a new one continues to be shown on the right in the web browswer,
  even when other cells get executed or rendered.  ``scratch.ipynb`` stores the
  ``always-show`` flag with the metadata for the results of the executed cell.
  As above, the association between RST directive instances and
  ``scratch.ipynb`` cells is via the directive UUID.
* I may want to play with the code in cell in the rendered ``scratch.ipynb``
  notebook.  I can do this, then switch back to the vim buffer, and execute some
  command such as ``:PullCell`` from an ``.. icode`` directive that will find
  the associated text and output from ``scratch.ipynb`` and use this to replace
  the current content in the ``.. icode`` directive;
* From vim, I can ``:RestartNBKernel`` to restart the IPython notebook kernel
  and ``:RunAllAbove`` to run all code cells above the cursor, replacing the
  corresponding cells in ``scratch.ipynb`` and refreshing rendering.
