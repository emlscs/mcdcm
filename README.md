mcdcm v0.1
=====

Javascript DICOM viewer for Orthanc

First of all: I don't know how to use github.
Second of all: Dammit, I'm a doctor, not a programmer. And a Trekkie (obviously).

This little thing I made (I call it Javascript, but all the files are PHP... how's that possible?) is based on Sébastien Jodogne's awesome Orthanc Dicom server (http://www.orthanc-server.com).

Basically, this is what happens:
- get the series-UUID in Orthanc that you want to view (easiest way is to look at the address bar in Orthanc Explorer)
- give a series-UUID (using HTTP-GET) of a series you want to view to viewer.php
- viewer.php will use the series/{id}/instances to retrieve the list of instance-UUIDs and corresponding instanceNumbers
- viewer.php will send the instance-UUIDs and corresponding instanceNumbers to 4 web-workers (bgLoadOdd.js, bgLoadOddTwo.js, bgLoadEven.js and bgLoadEvenTwo.js)
- the web-workers will call on pngToGray.php with the instance-UUIDs one by one
- pngToGray.php will retrieve the 16-bit PNG from orthanc using /instances/{id}/uint-16
- if the PNG file dimension is greater than 1000pixels in any direction, it is scaled using exec(imagemagick). If not for this, imagemagick is not required. GD2 is not compatible with 16-bit PNGs.
- pngToGray.php will parse the PNG and extract the compressed + filtered data in the IDATs, and perform decompression and unfiltering
- pngToGray.php will return a gzip compressed base 64-encoded string containing the image data to the web-workers
- web-workers will return the image data to viewer.php, which will keep it in an array
- viewer.php will uncompress the image data (using jsxcompressor http://jsxgraph.uni-bayreuth.de/wp/jsxcompressor/)
- viewer.php will reduce the 12-bit grayscale to 8-bit (HTML5 canvas compatible) grayscale and draw it in the canvas
- viewer.php catches the mouse events to scroll, pan and change windowing (zooming not implemented yet)

At the moment, performance is not that good. On a local network with server (Xeon E5-1603 @ 2.8GHz, 8Gb, Win7Pro64, Apache2.4, PHP5.5 TS) load times 
-- CT brain (512x512, 30 slices): time to first image 7s. time to completely load ~50s
-- AXR (downscaled 512 x 620, 1 slice): time to first image 8s. time to completely load 8s

Requirements:
- tested on Apache 2.4/PHP5.5
- imagemagick as exec command (I couldn't get imagick PHP extension to work...)
- tested on Safari and Chrome
