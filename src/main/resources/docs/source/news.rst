.. _NewIn114:

New or revised in version 2.0.2
===============================

A summary of major aspects:

 - **Text and OCR features** are now implemented using the Java library Tess4J (current latest version based on
   **Tesseract 4.x**). All Tesseract options are available at the Java API level.

 - **Transparency** Images with transparent areas (called masks) are now supported for search (ignoring the parts
   of the rectangles being transparent). Features will be available to create masks and masked images.

 - **App class revised** the implementation is now more robust and has some enhancements.

 - **setup no longer needed** You just download the IDE or API pack and simply start using it.

 - With the **IDE for Jython and/or JRuby** support an additional download is needed, since JavaScript support is not yet fully available.

 - **the IDE is a little bit revised** and got an **experimental recorder feature**

`Visit the new SikuliX Download page, where you can get more info and the needed stuff <https://raiman.github.io/SikuliX1/downloads.html>`_

.. note::

    ... **for Linux users** it is strongly recommended, to :ref:`first look here<newslinux>`.

Revision of the image find API
------------------------------

To support all features both for image and text search, the text search now has its own function set,
whose behavior is the same as for image search, except, that a given text is directly taken as the text to be
searched and not checked for a possible image file.

The old behavior of ``find(someText)`` is now switched off in the standard, but can be switched on
by intention (``Settings.SwitchToText``). But if switched on, the ``ImageMissing`` feature is not available.
In doubt it is recommended, to change the function calls to their text equivalents, if text search is the intention.

**How to read the following tables:**

In the 1st column you have the function names to be used as ``function(parameter1, parameter2, ...)``. About more
details :ref:`see the function description itself<FindinginsideaRegionandWaitingforaVisualEvent>`.

In all cases, these functions return a :py:class:`Match` object as result, which is ``null/None`` if not found.

 - **repeats-search** ``yes`` means, that the search is continued until either the image/text appears in the search region
   or until the standard/given waiting time is exceeded (which means a FindFailed). ``no`` means, that only one search is
   performed with the content of the search region at that moment (meaning either found or not found).

 - **throws-FindFailed** ``yes`` means, that in case of not found (also for elapsed waiting time), a FindFailed
   exception is thrown, which leads to script/program termination, if not handled somehow. ``no`` means, that the function
   silently returns ``null/None``, which must be checked if relevant.
   :ref:`About handling FindFailed cases read more here<ExceptionFindFailed>`.

==================   ==================   =====================
**image-function**   **repeats-search**   **throws-FindFailed**
  find                  no                  yes
  wait                  yes                 yes
  exists                yes                 no
  has                   yes                 no
  waitVanish            yes                 no
  findChanges           no                  no
==================   ==================   =====================

**has** is a convenience wrapper for exists and intended to be used in logical expressions (if, while, ...) since it
returns true in case of found and false otherwise. To get the match in case of found use ``getLastMatch()``.

**findChanges** The feature *look-for-changes* is already known from the ``observe`` feature ``onChange``.
The new find variant compares two images (must have exactly the same size) and returns a list of the areas,
that are different in the second image (for details see the function description in class Finder).

Revision of the text find API
-----------------------------

The equivalent functions that definitely look for the given text are named ``xxxText``.
There is also a shortcut set named only ``xxxT``. This might be helpful, when migrating to the new function set
for text search: use the xxxT version for replacements and xxxText for fresh implementations - or vice versa.

**Be aware** other than the image search functions, the text search functions search from top left to bottom right.
So if there is more than one possible match in a region, always the top left match is found.
With image search it is still so, that it cannot be foreseen, which of the possible matches is returned as the result.
In doubt you have to use the functions, that return all matches in a region and then filter the result to your needs.

==================   ==================   =====================
**text-function**    **repeats-search**   **throws-FindFailed**
  findText             no                   yes
  waitText             yes                  yes
  hasText              no                   no
  existsText           yes                  no
  waitVanishText       this function is     not available yet
  findAllText          no                   no
==================   ==================   =====================

And there are new text functions, that only search once and do not throw findFailed:
 - ``findWord("a word")`` looks for a word, whose text can be a regular expression
 - ``findLine("some text")`` looks for and returns the line, that contains the text (might be a regular expression)

.. note::

        **Regular expressions for text search** is not yet implemented.

:ref:`For details read about Text and OCR <textandocr>`.

Revision of the findAll feature
-------------------------------

All findAll variants only search once and do not throw a FindFailed. If nothing was found, you have to check the result
for being empty (not found) or not being empty. How to do that, depends on the function variant.

As with the find functions there are also findAllText and findAllT for the text only findAll functions. The resulting
list contain the matches in order top left to bottom right.

The function findAll returns a so called ``Match-Iterator`` (Java: Iterator<Match>), that reveals its content by the two
methods ``hasNext()`` (true if still some matches available or false) and ``next()`` (return the next match in the row).
The function ``next`` is greedy in that it removes the returned match from the internally managed list of matches.
So if you wanted a list, you always had to insert a step, that collects the matches into a list first. The order of the
resulting matches is still not predictable. They have to be sorted if needed.

Now there are function variants, that ``return match lists`` instead (Java: List<Match>). For empty or not you have to check
the size/length of the list, that is 0 when empty (not found)::

    # these return an Iterator of matches
    result = findAll("image")
    result.hasNext() # True or False
    result.next() # the next match or null/None if no more match

    # these return a list of matches
    result = findAllList("image") # equivalent to findAll()
    result = getAll("image") # shortcut for findAllList()
    result = findAllByRow() # sorted along rows
    result = findAllByColumn() # sorted along columns
    result = findAllText("some text") # or findAllT()
    result = findWords("a word") # like findWord() but returns all matches in the region
    result = findLines("some text") # like findLine() but returns all matches in the region

    # these return a list of matches (words or lines) in the region top left to bottom right
    result = findWords();
    result = findLines();

    # these return a list of words or lines in the region top left to bottom right (no match info)
    result = textWords();
    result = textLines();

    # to retrieve the found text from a text match
    theLineMatch = textLines[0]
    theLine = theLineMatch.getText()

For details see the function descriptions itself:

    - :ref:`image related <FindinginsideaRegionandWaitingforaVisualEvent>`.
    - :ref:`text/OCR related <textandocr>`

**Be Aware** Since version 1.1.2 there are also functions, that search for more than one image at the same time ::

    # find the best matching pattern of the given list of patterns
    result = findBest(pattern, pattern, pattern, ...) # var-arg parameterlist
    result = findBestList(List<patterns>) # a list of patterns

    # find all matching patterns in the list
    result = findAny(pattern, pattern, pattern, ...) # var-arg parameterlist
    result = findAnyList(List<patterns>) # a list of patterns

For details :ref:`see the function description itself<FindMoreThanOneImage>`.

Revision of the text and OCR feature
------------------------------------

The features are still supported by the library ``Tesseract OCR``. Until version 1.1.3 the usage of the library
was implemented via a C++ interface and the available features based on Tesseract 2.x have not changed for the last 6 years.

Now the Java library ``Tess4j`` is used, that allows to use the Tesseract features at the Java level. Internally it
depends on ``Tesseract 4.x``, that has major improvements in flexibility and accuracy.
Additionally you will have full access to all options available with Tesseract.

If you want to know anything about the features available through Tess4J/Tesseract, you have to dive into
the details on the respective home pages of the packages.

 - `Tess4J <http://tess4j.sourceforge.net>`_
 - `Tesseract <https://github.com/tesseract-ocr/tesseract>`_

.. note::

          The documentation of these packages is in a very basic stage with not much structure and
          little information that focuses on usage. So to dive into these docs only makes sense, if you really want to do
          special things or are not satisfied with the results you get with the SikuliX features.

**Request bugs for new features and/or revision/augmentation of existing ones are welcome**

**For detailed information and usage examples** :ref:`look here<textandocr>`.

Using images with transparent parts (masked images)
---------------------------------------------------

SikuliX now accepts images (PNG format only) that are partly transparent. This allows to search for images
on varying backgrounds and/or for images, whose content partly varies and should be ignored.

This is possible with a variant of the ``OpenCV matchTemplate()`` feature, that allows to specify a mask
as additional parameter. This mask has the same size in pixels as the image itself. All pixels that have a 0 in the mask
are ignored in the search.

SikuliX internally creates a mask from the transparency information in the image. All pixels having a transparency
of 100% (opacity is 0%) are ignored (will be 0 in the mask).

Additionaly the :py:class:`Pattern` now has features to specify images with the same size in pixels as the base image
with black areas, that are interpreted as transparent areas to create a mask accordingly.

SikuliX does not have features yet to create images with transparency nor mask images with black areas. To create such
images you have to use respective tools from the world of image handling or photo editing.

**Example on Mac** Open an image in the Preview app. Use one of the selection tools to select an area that should be
fully transparent and simply click backspace to *delete* the area. These steps might be repeated.
The image will automatically be a PNG file with an alpha-channel (the transparency information).

**There is a tutorial**: :ref:`Working with masked images (ignoring parts of the image) <tutorialMasking>`

App class revised
-----------------

For details look into the function descriptions at :py:class:`App`.

 - ``App.focus`` and ``App.close()`` now check, wether the app is still running
   (might have been closed manually meanwhile or simply crashed)

 - there is a new ``App.closeByKey()``, that tries to get the app to front and then use the system specific key
   combination (``Alt-F4, cmd-Q, ctrl-Q``) to gracefully close the application, which might not always be accomplished
   using the normal ``App.close()``.

 - ``App.open()`` and ``App.close()`` now have an optional parameter, to specify the maximum number of seconds
   to wait for the completion of the request.

 - ``App.toString()`` (scripting: ``print myApp``) now checks the internal state of the app and hence shows its
   actual state.

 - the logging verbosity is reduced to error situations, but can be raised with ``Debug.on(3)``.

New and revised features in the IDE
-----------------------------------

 - It is now possible to :ref:`run parts of a script <RunOnlyParts>`

