delayplayer
===========

![alt tag](https://raw.github.com/frisnit/delayplayer/master/delayplayer.png)

A collection of scripts to delay live streaming radio services for synchronising with different time zones. This is all very beta and very much a proof of concept.

See http://www.frisnit.com/2012/03/12/delayplayer-radio-4-synced-to-your-timezone/ for full details

The site can be seen at http://cdn.frisnit.com/delayplayer/ although the streaming functionality will cease at the end of February 2014.

The system consists of two main parts:

Ingest
------
> This connects to the remote AAC audio streams, transcodes them to MP3 using ffmpeg and saves them as timestamped files in chunks of about ten minutes duration. These processes are started by cron (start.sh) so that they're restarted automatically if they drop out (which they do occasionally). There are three elements to the injest chain that are piped together:

    save_from_pipe.pl : connect to remote stream and output binary AAC data to STDOUT
    r2_to_pipe.pl     : each of these three files do the same thing and contain the same code
    r1_to_pipe.pl     : but haven't been combined in to a single file yet
    
    ffmpeg            : take input from STDIN, transcode and output binary MP3 data to STDOUT
    save_from_pipe.pl : take input from STDIN and save in timestamped files to disk
    
    example (all errors piped to /dev/null):
    
    # start radio 1
    /home/ec2-user/delayplayer/transcoder/stream_to_pipe.pl --service bbc_radio_one | 
        /usr/local/bin/ffmpeg -i pipe:0 -f mp3 -acodec libmp3lame -b:a 56k -ar 32000 pipe:1 2> /dev/null | 
        /home/ec2-user/delayplayer/transcoder/save_from_pipe.pl --path /home/ec2-user/delayplayer/transcoder/radio_one/  > /dev/null 2>&1 &

    
    

> Files older than 24 hours are deleted by a cron job that runs every hour. All the cron jobs used in this project can be found in crontab.txt.

Playout
-------
> An Icecast server operating in relay mode (config file icecast_minimal.xml) maintains a number of streams endpoint for each station and timezone. Each of these connects to an instance of a lightweight server ('streamer') that streams the timestamped chunks from the appropriate point in the past (on port 2345 in this example). This process is started by cron (start_streamer.sh) so that it's restarted automatically if it drops out (which it does occasionally). You'll need to compile streamer.c on your system to create the 'streamer' binary.



There are a few other supporting systems:

Now playing
-----------
> The front page HTML is generated from the radio schedule by a cron job that fires off every minute or so

    update_now_playing.php : gets the data and writes it to disk
    source_stations.php    : contains details of the stations and schedule endpoints

Monitoring
----------
> The system status is monitored by checking the injest files are being created and that the Icecast endpoints are still available. This is performed by a simple .php script run from cron every five minutes

    monitor_all.php


Front end
---------
The front end HTML and Flash player files are contained in the /html directory. Make this your server root. The .htaccess file performs a basic redirect to the mobile view.

Dependencies
------------
You'll also have to install Icecast and ffmpeg with AAC and MP3 support.
    
  

