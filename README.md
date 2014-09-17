EyeTribe Toolbox for Matlab
===========================

version 0.0.2 (17-Sep-2014)


ABOUT
-----

The EyeTribe Toolbox for Matlab is a set of functions that can be used
to communicate with eye trackers manufactured by [the EyeTribe](https://theeyetribe.com/). The
communication process is not direct, but goes via a sub-server that
receives input from Matlab (when the functions from this toolbox are
called), and then sends commands to the actual EyeTribe server.

This setup is rather odd, but it is the most elegant solution that I
could come up with to get around the problem of Matlab not having
decent multithreading functionality. This functionality is required
for running a heartbeat Thread (which keeps the connection with the
EyeTribe alive), and another Thread to monitor samples (and write these
to a log file). Similar results might be obtained by using callback
functions within Matlab's TCP/IP framework, but that approach causes
timing errors that extent into other domains: timing issues when
using PsychToolbox's `WaitSecs` function, and background processes
in Matlab screwing up all sorts of other timing sensitive processes.

So, out of lazine... Err... Out of a well-planned timing management
effort to avoid time loss by re-inventing the wheel, I simply used
[PyTribe](https://github.com/esdalmaijer/PyTribe) in a short Python script (see the `python_source` folder for
the source) to compile a Windows executable, which should be run
before you run your Matlab script.

The calibration routine is based on the [PsychToolbox](https://psychtoolbox.org/HomePage) for Matlab,
and requires an active window to be passed to it. This assures that
you are free to calibrate the tracker at any given moment in your
experiment, without having any external calibration routine battle
with your experiment for control of the active display.

If you do not want to calibrate using the PsychToolbox, you can still
use the EyeTribe Toolbox for Matlab, by simply NOT calling the
`eyetribe_calibrate` function. Please do note that you should then
calibrate the system with your own means, e.g. by using the EyeTribe's
own GUI (`C:\Program Files (x86)\EyeTribe\Client\EyeTribeWinUI.exe`)
*before* starting any software that calls upon the EyeTribe Toolbox
for Matlab.

DOWNLOAD
--------

1) Go to: [https://github.com/esdalmaijer/EyeTribe-Toolbox-for-Matlab](https://github.com/esdalmaijer/EyeTribe-Toolbox-for-Matlab)

2) Press the Download ZIP button, or click this [direct link](https://github.com/esdalmaijer/EyeTribe-Toolbox-for-Matlab/archive/master.zip).

3) Extract the ZIP archive you just downloaded.

4) Copy the folder `EyeTribe_for_Matlab` to where you want it to be
(e.g. in your *Documents* folder, under *MATLAB*).

5) In Matlab, go to *File -> Set Path -> Add folder* and select
the folder you copied at step 4.

Alternatively, place the following code at the start of your experiment:

~~~ .matlab
% assuming you placed the EyeTribe_for_Matlab directly under C:
addpath('C:\EyeTribe_for_Matlab')
~~~


USAGE
-----

1. Start EyeTribe Server
	`C:\Program Files(x86)\EyeTribe\Server\EyeTribe.exe`

2. Start `EyeTribe_Matlab_server.exe`.

3. Run your Matlab script, e.g. the one below:

~~~ .matlab
% don't bother with vsync tests for this demo
Screen('Preference', 'SkipSyncTests', 1);

% initialize connection
[success, connection] = eyetribe_init('test');

% open a new window
window = Screen('OpenWindow', 2);

% calibrate the tracker
success = eyetribe_calibrate(connection, window);

% show blank window
Screen('Flip', window);

% start recording
success = eyetribe_start_recording(connection);

% log something
success = eyetribe_log(connection, 'TEST_START');

% get a few samples
for i = 1:60
    pause(0.0334)
    [success, x, y] = eyetribe_sample(connection);
    [success, size] = eyetribe_pupil_size(connection);
    disp(['x=' num2str(x) ', y=' num2str(y) ', s=' num2str(size)])
end

% log something
success = eyetribe_log(connection, 'TEST_STOP');

% stop recording
success = eyetribe_stop_recording(connection);

% close connection
success = eyetribe_close(connection);

% close window
Screen('Close', window);
~~~

