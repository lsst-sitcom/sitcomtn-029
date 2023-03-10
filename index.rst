..
  Technote content.

  See https://developer.lsst.io/restructuredtext/style.html
  for a guide to reStructuredText writing.

  Do not put the title, authors or other metadata in this document;
  those are automatically added.

  Use the following syntax for sections:

  Sections
  ========

  and

  Subsections
  -----------

  and

  Subsubsections
  ^^^^^^^^^^^^^^

  To add images, add the image file (png, svg or jpeg preferred) to the
  _static/ directory. The reST syntax for adding the image is

  .. figure:: /_static/filename.ext
     :name: fig-label

     Caption text.

   Run: ``make html`` and ``open _build/html/index.html`` to preview your work.
   See the README at https://github.com/lsst-sqre/lsst-technote-bootstrap or
   this repo's README for more info.

   Feel free to delete this instructional comment.

:tocdepth: 1

.. Please do not modify tocdepth; will be fixed when a new Sphinx theme is shipped.

.. sectnum::



Introduction
============

Adding a filter or disperser to LATISS is a multi-step procedure that spans the DM and T&S subsystems.
This technote is a list of things to do in order to have the data be taken with the correct metadata, as well as what it takes to get the files ingested on the summit.

.. important::

   The most time critical step is updating filters.py (see below) and having it be built in a regular weekly that is accessible from the summit environment.
   The DM weeklies are built on Wednesday nights and are generally available Thursday morning, the T&S container then builds on top of that, adding several hours.
   Performing last-minute filter additions is high risk and puts strain on personnel. 
   Therefore, any new filter or grating names should be added as early as possible, preferably weeks in advance.

.. note::

   A future update to this technote will include the steps required to ingest the data at USDF.



Procedure
=========

1. Make a ticket on the SUMMIT project that dictates which filter should be removed and which should be added.
   Use a previous ticket as a reference. 
   Roberto Tighe and Mario Rivera are trained in performing the physical filter change.

2. Create a DM ticket (Team = Telescope and Site) to update the config file in the `ts_config_latiss <https://github.com/lsst-ts/ts_config_latiss>`_ repository.
   Follow the TSSW development process.
   Never change the names of filters.
   Update the name and any focus offsets. 
   Use the previous configs as the reference for the correct values.

3. After the filter is installed, and before taking any images, update the CSC to use the tag (or branch) containing the new config.
   This needs to be done either inside the container (temporary) or by updating the cycle.env file, then rebuilding and redeploying the ATSpectrograph CSC (which is the proper way). 

4. Check that the filter is in the `filters.py file of obs_lsst <https://github.com/lsst/obs_lsst/blob/main/python/lsst/obs/lsst/filters.py>`_.
   If not, follow the DM development process of making a ticket, making the change, running the CI tests, and getting it reviewed and merged.
   The changes will be available in the next weekly.

5. Update the OODS container to use the new ``obs_lsst`` that contains the changes (if not done by a cycle deployment).

6.	Once everything built and deployed, the butler needs to have the registry updated to say the filter exists. So each butler repo (on each instance such as the test stands) need to run the following from the command line wherever the new ``obs_lsst`` is installed. At the summit, this can be done from inside a Nublado instance (and probably the other places too). Note that $OODS_REPO_PATH needs to be populated or replaced. On the summit it is ``/repo/LATISS``

   .. code-block::

		butler register-instrument $OODS_REPO_PATH lsst.obs.lsst.Latiss
		
7. The USDF embargo Butler auto ingestion service must also be restarted with the appropriate weekly, after registering the instrument in the embargo Butler repo (and also the main Butler repo).

8. Start taking images with the new filter and/or grating and verify ingestion works and that images appear on RubinTV.
		






.. Add content here.
.. Do not include the document title (it's automatically added from metadata.yaml).

.. .. rubric:: References

.. Make in-text citations with: :cite:`bibkey`.

.. .. bibliography:: local.bib lsstbib/books.bib lsstbib/lsst.bib lsstbib/lsst-dm.bib lsstbib/refs.bib lsstbib/refs_ads.bib
..    :style: lsst_aa
