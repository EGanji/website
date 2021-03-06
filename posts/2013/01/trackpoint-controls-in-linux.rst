.. link: 
.. description: 
.. tags: linux
.. wordpress_id: 130
.. date: 2013-01-24 14:15:50
.. slug: trackpoint-controls-in-linux
.. title: TrackPoint controls in Linux
.. description: In which I detail how to fix issues with TrackPoint speed and sensitivity on Linux.
.. comments: true

.. role:: bash(code)
    :language: bash

So it seems as if the most recent updates to Kubuntu 12.10 have resulted in a stable graphics experience on my NVidia optimus chipset. As a result, I can now live in penguin land 24/7.

However, transitioning from a fully Lenovo-supported OS (Windows) to a "we'll allow it" OS (Linux) presents it's own set of issues. One can mostly muddle through with enough Google-fu, but one thing that has always vexed me is configuring my TrackPoint (the "red eraser" that TPs are infamous for). The default pointer calibration settings exposed by KDE's system settings are pretty minimal - only allowing me to configure a handful of settings that would apply to both the TrackPoint as well as the touchpad. Not very fun. I set about digging, and found something that worked for my ThinkPad T530:

#. Install Sysfs: :bash:`sudo apt-get install sysfsutils`

#. Then, enter a root shell: :bash:`sudo su`

#. Observe the values of the two settings we care about: \

   .. code:: bash

    cat /sys/devices/platform/i8042/serio1/serio2/speed
    80
  
   and \
   
   .. code:: bash
    
    cat /sys/devices/platform/i8042/serio1/serio2/sensitivity
    110
      
#. To alter your TrackPoint's speed, \

   .. code:: bash

    echo -n 110 > /sys/devices/platform/i8042/serio1/serio2/speed
  
   Possible values fall within the the range of 0 - 255, inclusive.  Since the results are immediate, you can use trial and error to find a suitable speed.

#. To alter your TrackPoint's sensitivity, \

   .. code:: bash
  
    echo -n 260 > /sys/devices/platform/i8042/serio1/serio2/sensitivity
    
   Possible values fall within the the range of 0 - 255, inclusive.  Since the results are immediate, you can use trial and error to find a suitable sensitivity.
	
#. Once you've settled on the appropriate values, you'll need to make things permanent. Open your rc.local file and insert the following lines before the "`exit 0`" return statement: \

   .. code:: bash
    
    echo -n 160 > /sys/devices/platform/i8042/serio1/serio2/sensitivity
    echo -n 110 >  /sys/devices/platform/i8042/serio1/serio2/speed
  
   Be sure to replace my values with the ones you deduced in the previous two steps.


And that should do it!  There's a graphical tool for managing this, called configure_trackpoint, but it has a whole bunch of Gnome dependences. *shudders*
