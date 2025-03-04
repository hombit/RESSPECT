.. _preprocessing:

Feature Extraction
==================

The first stage in consists in transforming the raw data into a uniform data matrix which will subsequently be given
as input to the learning algorithm.

``resspect`` can handle FITS format data from the RESSPECT project, csv data from the Photometric LSST Astronomical Classification Challenge (`PLAsTiCC <https://zenodo.org/record/2539456#.Xrsk33UzZuQ>`_)  and text-like data from the SuperNova Photometric Classification Challenge (`SNPCC <https://arxiv.org/abs/1008.1024>`_).


Load 1 light curve: 
-------------------

For RESSPECT
^^^^^^^^^^^^

In order to fit a single light curve from the RESSPECT simulations you need to have its identification number. This information is stored in the header SNANA files. One of the possible ways to retrieve it is:

.. code-block:: python
    :linenos:

    >>> import io
    >>> import pandas as pd
    >>> import tarfile

    >>> path_to_header = '~/RESSPECT_TRAIN_HEAD.csv.gz'
    >>> header = pd.read_csv(path_to_header)

    # get keywords
    >>> header.keys()
    Index(['objid', 'redshift', 'type', 'code', 'sample'], dtype='object')

    # check the first chunks of ids and types
    >>> header[['SNID', 'type']].iloc[:10]
       SNID     TYPE
    0   3228  Ibc_V19
    1   2241      IIn
    2   6770       Ia
    3    302      IIn
    4   7948       Ia
    5   4376   II_V19
    6    337   II_V19
    7   6017       Ia
    8   1695       Ia
    9   1660   II-NMF  

Choose one SNID to work with:

.. code-block:: python
    :linenos:
    >> snid = header['SNID'].values[4]


Now that you have selected one object, you can read its light curve using the `LightCurve class <https://resspect.readthedocs.io/en/latest/api/resspect.LightCurve.html#resspect.LightCurve>`_ :


.. code-block:: python
    :linenos:

    >>> from resspect.fit_lightcurves import LightCurve

    >>> path_to_lightcurves = '~/RESSPECT_TRAIN_LIGHTCURVES.tar.gz'

    >>> lc = LightCurve()
    >>> lc.load_resspect_lc(photo_file=path_to_lightcurves, snid=snid)

    # check light curve format
    >>> lc.photometry
              mjd band      flux   fluxerr        SNR
    0     53058.0    u  0.138225  0.142327   0.971179
    1     53058.0    g -0.064363  0.141841  -0.453768 
    ...       ...  ...       ...       ...        ...
    1054  53440.0    z  1.173433  0.145918   8.041707
    1055  53440.0    Y  0.980438  0.145256   6.749742

[1056 rows x 5 columns]


For PLAsTiCC:
^^^^^^^^^^^^^

Similar to the case presented below, reading only 1 light curve from PLAsTiCC requires an object identifier. This can be done by:

.. code-block:: python
    :linenos:

    >>> from resspect.fit_lightcurves import LightCurve
    >>> import pandas as pd

    >>> path_to_metadata = '~/plasticc_train_metadata.csv'
    >>> path_to_lightcurves = '~/plasticc_train_lightcurves.csv.gz'
    
    # read metadata for the entire sample
    >>> metadata = pd.read_csv(path_to_metadata)

    # check keys
    >>> metadata.keys()
    Index(['object_id', 'ra', 'decl', 'ddf_bool', 'hostgal_specz',
           'hostgal_photoz', 'hostgal_photoz_err', 'distmod', 'mwebv', 'target',
           'true_target', 'true_submodel', 'true_z', 'true_distmod',
           'true_lensdmu', 'true_vpec', 'true_rv', 'true_av', 'true_peakmjd',
           'libid_cadence', 'tflux_u', 'tflux_g', 'tflux_r', 'tflux_i', 'tflux_z',
           'tflux_y'],
         dtype='object')
    
    # choose 1 object
    >>> snid = metadata['object_id'].values[0]

    # create light curve object and load data
    >>> lc = LightCurve()
    >>> lc.load_plasticc_lc(photo_file=path_to_lightcurves, snid=snid)
    

For SNPCC:
^^^^^^^^^^

The raw data looks like this:

.. literalinclude:: images/DES_SN729076.DAT
 :lines: 1-61

You can load this data using:

.. code-block:: python
   :linenos:

   >>> from resspect.fit_lightcurves import LightCurve

   >>> path_to_lc = 'data/SIMGEN_PUBLIC_DES/DES_SN729076.DAT'

   >>> lc = LightCurve()                        # create light curve instance
   >>> lc.load_snpcc_lc(path_to_lc)             # read data


Fit 1 light curve:
-----------

Once the data is properly loaded, the photometry can be recovered by:


.. code-block:: python
   :linenos:

   >>> lc.photometry                            # check structure of photometry
             mjd band     flux  fluxerr   SNR
    0	56194.012	g	13.090	6.204	2.11	99.000	5.000
    1	56194.016	r	-4.680	3.585	-1.31	99.000	5.000
    ...	...	...	...	...	...	...	...
    75	56317.051	i	173.200	7.661	22.60	21.904	0.049
    76	56318.035	z	141.000	13.720	10.28	22.127	0.111


You can now fit each individual filter to the parametric function proposed by
`Bazin et al., 2009 <https://arxiv.org/abs/0904.1066>`_ in one specific filter.

.. code-block:: python
   :linenos:

   >>> rband_features = lc.fit_bazin('r')
   >>> print(rband_features)
  [514.92432962  -5.99556655  40.59581991  40.03343317   3.74307339]

The designation for each parameter are stored in:

.. code-block:: python
   :linenos:

   >>> lc.bazin_features_names
   ['a', 'b', 't0', 'tfall', 'trise']

It is possible to perform the fit in all filters at once and visualize the result using:

.. code-block:: python
   :linenos:

   >>> lc.fit_bazin_all()                            # perform Bazin fit in all filters
   >>> lc.plot_bazin_fit(save=True, show=True,
   >>>                   output_file='plots/SN' + str(lc.id) + '_flux.png')   # save to file

.. figure:: images/SN729076_flux.png
   :align: center
   :height: 480 px
   :width: 640 px
   :alt: Bazing fit to light curve. This is an example from SNPCC simulations.

   Example of light curve from SNPCC simulations.


This can be done in flux as well as in magnitude:

.. code-block:: python
    :linenos:

    >>> lc.plot_bazin_fit(save=False, show=True, unit='mag')

.. figure:: images/SN729076_mag.png
   :align: center
   :height: 480 px
   :width: 640 px
   :alt: Bazing fit to light curve. This is an example from SNPCC data.

    Example of light from SNPCC data.


Ocasionally, it is necessary to extrapolate the fitted light curve to a latter epoch -- for example, in case we want to estimate its magnitude at the time of spectroscopic measurement (details in the `time domain preparation section <https://resspect.readthedocs.io/en/latest/prepare_time_domain.html>`_ ).

Before deploying  large batches for pre-processing, you might want to visualize how the extrapolation behaves for a few light curves. This can be done using:

.. code-block:: python
    :linenos:

    >>> # define max MJD for this light curve
    >>> max_mjd = max(lc.photometry['mjd']) - min(lc.photometry['mjd'])
    
    >>> lc.plot_bazin_fit(save=False, show=True, extrapolate=True, 
                          time_flux_pred=[max_mjd+3, max_mjd+5, max_mjd+10])


.. figure:: images/SN729076_flux_extrap.png
   :align: center
   :height: 480 px
   :width: 640 px
   :alt: Bazing fit to light curve. This is an example from SNPCC data.

    Example of extrapolated light from SNPCC data.


Processing all light curves in the data set
-------------------------------------------

There are 2 way to perform the Bazin fits for all three data sets. Using a python interpreter,


For RESSPECT:
^^^^^^^^^^^^^

.. code-block:: python
   :linenos:

   >>> from resspect import fit_resspect_bazin

   >>> photo_file = '~/RESSPECT_TRAIN_LIGHTCURVES.csv.gz' 
   >>> header_file = '~/RESSPECT_TRAIN_HEAD.csv.gz'
   >>> output_file = 'results/RESSPECT_Bazin_train.dat'     

   >>> sample = 'train'       

   >>> fit_resspect_bazin(photo_file, header_file, output_file, sample=sample)


For PLAsTiCC:
^^^^^^^^^^^^^

.. code-block:: python
   :linenos:

   >>> from resspect import fit_plasticc_bazin

   >>> photo_file = '~/plasticc_train_lightcurves.csv' 
   >>> header_file = '~/plasticc_train_metadata.csv.gz'
   >>> output_file = 'results/PLAsTiCC_Bazin_train.dat'            

   >>> sample = 'train'

   >>> fit_plasticc_bazin(photo_file, header_file, output_file, sample=sample)


For SNPCC:
^^^^^^^^^^

.. code-block:: python
   :linenos:

   >>> from resspect import fit_snpcc_bazin

   >>> path_to_data_dir = 'data/SIMGEN_PUBLIC_DES/'            # raw data directory
   >>> output_file = 'results/Bazin.dat'                              # output file

   >>> fit_snpcc_bazin(path_to_data_dir=path_to_data_dir, features_file=output_file)



The same result can be achieved using the command line:

.. code-block:: bash
    :linenos:

    # for RESSPECT or PLAsTiCC
    >>> fit_dataset.py -s <dataset_name> -p <path_to_photo_file> 
             -hd <path_to_header_file> -sp <sample> -o <output_file> 

    # for SNPCC
    >>> fit_dataset.py -s SNPCC -dd <path_to_data_dir> -o <output_file>
