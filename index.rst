##############################
LATISS Filter Change Procedure
##############################

.. abstract::

This document explains the required procedures when replacing or adding a *new*, previously uninstalled, filter or grating to LATISS.
These steps are required before any data is taken, in order to guarantee the data is correctly handled by the acquisition and ingestion system.

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






Introduction
============

Replacing a filter or disperser in LATISS requires a few steps to ensure that the instrument is properly configured and that data ingestion is working correctly.
When adding a new, previously uninstalled filter or disperser, there are a few additional steps that span both DM and T&S subsystems.
Some of these steps may take some time to be ready to be rolled out to the summit and should be done well in advance to prevent delaying operations

.. important::

   When adding a new, previously uninstalled filter, the most time critical step is updating the `filters definition file <https://github.com/lsst/obs_lsst/blob/main/python/lsst/obs/lsst/filters.py>`_ (see below) and having it be built in a regular weekly that is accessible from the summit environment.
   The DM weeklies are built on Wednesday nights and are generally available Thursday morning, the T&S container then builds on top of that, adding several hours.
   Performing last-minute filter additions is high risk and puts strain on personnel. 
   Therefore, any new filter or grating names should be added as early as possible, preferably weeks in advance.


Procedure
=========

#. Make a ticket on the SUMMIT project that dictates which filter should be removed and which should be added.
   Use a previous ticket as a reference. 
   Roberto Tighe and Mario Rivera are trained in performing the physical filter change.

#. Create a DM ticket (Team = Telescope and Site) to update the config file in the `ts_config_latiss <https://github.com/lsst-ts/ts_config_latiss>`_ repository.
   Use the previous configs as the reference for the correct values of filter names and focus offsets. 
   Follow the `TSSW development process <https://tssw-developer.lsst.io/work_management/development_workflow.html#development-workflow-release-process>`_ to tag and release a new version of the `ts_config_latiss <https://github.com/lsst-ts/ts_config_latiss>`_ package once the changes are merged.

#. If you are installing a new, previously unused filter, you will also need to open a PR to update the the `filters definition file <https://github.com/lsst/obs_lsst/blob/main/python/lsst/obs/lsst/filters.py>`_. 
   Follow the DM development process of making a ticket, making the change, running the CI tests, and getting it reviewed and merged. 
   The changes will be available in the next DM weekly.
   If you are installing a known filter, you can skip directly to step 10.

#. Using the new DM weekly with the changes to the `filters definition file <https://github.com/lsst/obs_lsst/blob/main/python/lsst/obs/lsst/filters.py>`_, register the new filter definitions in all of the different standard butler repositories using the command below. 
   Note that $OODS_REPO_PATH needs to be replaced with the appropriate path depending on where you are running the command. 

   .. code-block:: bash

    butler register-instrument $OODS_REPO_PATH lsst.obs.lsst.Latiss

   Currently, the repo paths are:

   - **At USDF**: /repo/embargo and /repo/main
   - **At Summit**: /repo/LATISS
   - **At BTS**: /repo/LATISS
   - **At TTS**: /repo/LATISS

#. Update the cycle build. Change the weekly version of summit_utils, summit_extras, atmospec, spectractor, lsst-sqre to use the new DM weekly.

#. Rebuild the following containers using the `current cycle build on jenkins <https://ts-cycle-build.lsst.io/user-guide/user-guide.html#fig-jenkins-build-with-parameters>`_

   - deploy_lsstsqre
   - build_scriptqueue
   - rapid_analysis
   - build-sciplat
   - build-sciplat-recommended
   - build-oods

#. Once the container builds are complete, redeploy the ATOODS container and rubinTV pods. 

#. The USDF embargo Butler auto ingestion service must also be restarted with the appropriate weekly, after registering the instrument in the embargo Butler repo (and also the main Butler repo).

#. After the filter is physically installed, and before taking any images, update the ATSpecgtrograph CSC to use the tag (or branch) containing the new config.
   This needs to be done either inside the container (temporary) or by updating the cycle.env file, then rebuilding and redeploying the ATSpectrograph CSC (which is the proper way). 

#. Once the new version of the ATSpectrograph CSC is deployed you are ready to enable the spectrograph and take a few test images with the new configuration.

#. During the filter change process, it is possible the grating stage itself was moved out of its nominal position, so be sure to start by running the `latiss checkout procedure <https://obs-ops.lsst.io/AuxTel/Standard-Operations/Daytime-Operations/Daytime-Checkout.html#auxtel-daytime-checkout-latiss-checkout-py>`_ to check the position of this stage.		

#. Check that the images are properly ingested in RubinTV by looking for the filter and grating and ensure that the values are correct. 
   If the values for filter and grating are correct, you are finished. 

.. Add content here.
.. Do not include the document title (it's automatically added from metadata.yaml).

.. .. rubric:: References

.. Make in-text citations with: :cite:`bibkey`.

.. .. bibliography:: local.bib lsstbib/books.bib lsstbib/lsst.bib lsstbib/lsst-dm.bib lsstbib/refs.bib lsstbib/refs_ads.bib
..    :style: lsst_aa