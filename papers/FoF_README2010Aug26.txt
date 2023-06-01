JMC Notes 2010 Aug 26
pulse finding code from ~2003
code is in pulse.match.tar.gz and was set up for
	compiling under linux
A. pulse.match:
	original code from mid-1990s did hierarchical smoothing + thresholding
	1.  find all samples above threshold and report
	2.  add adjacent samples and decimate by 2 
	3.  find all samples above threshold with threshold raised by sqrt(2)
	4.  repeat steps 2 and 3 up to requested number of levels
		(i.e. we often used 8 levels, so the maximum number of
		smoothed samples is 2^{8-1} = 128) 
	
	note that smoothing filters in the different levels have
	coefficients that follow Pascal's triangle:

	1 -> 1 1 -> 1 2 1 -> 1 3 3 1  etc.

	note also code reports _samples_ that are above threshold;
	in general, there can be multiple samples per _event_.
	So the reported values cannot be interpreted as events.

B. Friends of friends (referred to as "sift" in the code):
	It could be called friends of friends or called a "cluster"
		algorithm since it identified clusters of events
		that are part of the same event.
	This was written by me in 2003 to address the samples/event
	issue of A and used for the Crab GP analysis in
	Cordes et al. 2004, ApJ

	The algorithm works like this:
	1. Set a fairly low threshold, like 3 sigma.
	2. Find all samples that are above this threshold.
	   For these samples, find those that are contiguous
	   or separated by no more than ncluster samples from each
	   other  (i.e. gaps are allowed between samples up to
	   ncluster in length).  The code currently has ncluster=2
	   hardwired. This can/should be made an input variable.   

	   For all samples that are so identified as part of the same
           burst, find sum of all the samples and the mean sample number
	   of all the samples (weighted by amplitude).   Also find the
	   maximum amplitude of all the samples in the cluster and
	   find an effective width (in samples) that is the sum of all
	   samples divided by the maximum amplitude. 
	   These are reported along with the local off-pulse mean
	   and rms (these are calculated iteratively by excluding
	   all samples that are above threshold). 

	   All of this happens in zwrpul_sift.c.
	   The write statement to the output file (XXXX.sift.dat,
	   renamed in some of my analysis to pulse.list.YYYY) is:

	   fprintf(fd,"%2d %2d %2d %9d %8.2f %8.2f %8.2f %8.2f %5d %8.3f %8.3f\n",
                 ndm, nsubband,  ns, iimax,
                 amax,  pulse[imax].mean, pulse[imax].rms,
                 suma, nsum, wgp, tsum-iimax);

	   where 
		ndm = number of dispersion channel (e.g. if we were
		      doing lots of trial DMs in a search;  for the Crab
		      I used two DMs: zero and the Crab's DM)
		nsubband = subband number if we process separately
		ns = level of smoothing (if we were doing pulse.match;
	             this format is an extension of what was used for
		     pulse.match so for sift.dat ns=0 (no smoothing).
		iimax = sample number in file of maximum in cluster
		amax = maximum amplitude of cluster in sigma units
			(i.e. this is S/N = peak / off-pulse-rms)
		pulse[imax].mean = local off-pulse mean (we analyzed
			the time series in blocks and this is the block mean)
		pulse[imax].rms = rms in block
		suma = sum of all samples in cluster
		nsum = number of samples in cluster
		wgp = suma/amax = effective width of cluster in samples
		tsum-iimax = mean pulse time minus iimax (i.e. an asymmetric
			pulse would have a mean different from iimax)

	The fof algorithm worked very well on the Crab by finding
	just single clusters associated with any given giant pulse.
	This might be different with high-resolution data where the
	emission in a given pulse might comprise multiple peaks.
	How multiple peaks are reported will depend on ncluster,
	so there is some control.  You could also put a cap
	on wgp that any GP is allowed to have, though this 
	wouldn't be bullet proof.

Examples: 
	In pulse.list.B0531+21.52304.023 there is an event record:

	1  0  0    386118 11030.49   940.73    14.14 29969.84    77    2.717   -0.156
	This was the largest event we saw in one hour of 430 MHz data
	at Arecibo.  It has S/N = 11030.49.   There is a plot of
	this pulse in Cordes et al. 2004
