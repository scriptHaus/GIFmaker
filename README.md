## How to create a GIF from the command line on MacOS
(Unleash your inner Spielberg and create a custom GIF)

Looking to up your GIF game on your next team connect?  Not interested in using a sketchy website that may limit your creative talents and sell your data?  Follow these steps to create the perfect GIF to share with friends and colleagues.

This method blends a youtube clip with static images to create a customized GIF file.

*nb: this method requires the use of yt-dlp and ffmpeg already installed on your computer*

---
1. Search youtube videos for your preferred theme.  Filter results for Creative Commons licensing.

   <img src="https://raw.githubusercontent.com/scriptHaus/GIFmaker/main/screenshots/youtube_filters.png" alt="youtube filter selection" width=40% height=40%>

2. After locating your desired video, you will need to download the file in a suitable GIF size (likely 480x360 or 640x360 pixels).

   - List the available formats

        ```bash
        yt-dlp -F "https://www.youtube.com/watch?v=________"
        ```

   - Download the segment of the video (only, no audio needed) in the mp4 format along with a separate subtitles file

        ```bash
        yt-dlp -o "____.mp4" --download-sections "*0:10-0:30" --write-auto-subs --sub-lang en -f 230 "https://www.youtube.com/watch?v=_______"
        ```

3. Open the vtt file in a text editor and edit the file to match the timing of the segment of video.

   - An auto sub file includes the entire video dialog which you will need to edit

      <img src="https://raw.githubusercontent.com/scriptHaus/GIFmaker/main/screenshots/vtt_before.png" alt="VTT file before editing" width=40% height=40%>

   - Delete the unneccessary text and adjust the dialog timing to match the specific segment of video

      <img src="https://raw.githubusercontent.com/scriptHaus/GIFmaker/main/screenshots/vtt_after.png" alt="VTT file after" width=40% height=40%>

5. Use your favorite graphics program [Krita/Affinity Designer/Photoshop/etc] to create a series of images (jpgs or pngs) at the same size as the video (eg, 480x360 or 640x360)
   
6. Use ffmpeg to create a video slideshow of the images in your desired order <br> 
   *nb, adjust the image times and transition times in the vtt file as needed*

    ```bash
    ffmpeg \
    -loop 1 -t 2 -i image1.png \
    -loop 1 -t 2 -i image2.png \
    -loop 1 -t 2 -i image3.png \
    -loop 1 -t 2 -i image4.png \
    -loop 1 -t 2 -i image5.png \
    -filter_complex \
    "[0:v]fade=t=in:st=0:d=0.5,fade=t=out:st=1.5:d=0.5[v0]; \
    [1:v]fade=t=in:st=0:d=0.5,fade=t=out:st=1.5:d=0.5[v1]; \
    [2:v]fade=t=in:st=0:d=0.5,fade=t=out:st=1.5:d=0.5[v2]; \
    [3:v]fade=t=in:st=0:d=0.5,fade=t=out:st=1.5:d=0.5[v3]; \
    [4:v]fade=t=in:st=0:d=0.5,fade=t=out:st=1.5:d=0.5[v4]; \
    concat=n=5:v=1:a=0,format=yuv420p[v]" \
    -map "[v]" slides.mp4
    ```

5. Once you have the total timing of the slides, go back and re-download the specific segments of the youtube video to fit around the slideshow you've just created

    ```bash
    yt-dlp -o "opening.mp4" --download-sections "*0:10-0:20" -f 230 "https://www.youtube.com/watch?v=_______"
    ```

    ```bash
    yt-dlp -o "ending.mp4" --download-sections "*0:30-0:40"  -f 230 "https://www.youtube.com/watch?v=_______"
   ```

6. Re-encode the short clips, if necessary

    ```bash
    ffmpeg -i opening.mp4 -c:v libx265 -r 29.97 intro.mp4
    ```

    ```bash
    ffmpeg -i ending.mp4 -c:v libx265 -r 29.97 outro.mp4
    ```

7. Convert any additional static images into short clips

    ```bash
    ffmpeg -framerate 1 -i static_image.png -c:v libx265 -r 29.97 static_image.mp4
    ```

8. Combine all of the clips into one

    ```bash
    ffmpeg \
    -i intro.mp4 -i slides.mp4 -i outro.mp4 -i -i static_image.mp4 \
    -fps_mode vfr -filter_complex "[0:v:0][1:v:0][2:v:0][3:v:0]concat=n=4:v=1:a=0[outv]" \
    -map "[outv]" -c:v libx265 silent_movie.mp4
    ```

9.  Adjust the timing of the vtt file to match the combined video then merge the subtitles into the video file <br> 
   (h/t: https://bernd.dev/2020/04/adding-subtitles/)

    ```bash
    ffmpeg -i VTT_FILE_NAME.en.vtt temp.ass && \
    BITRATE=$(ffprobe -v error -select_streams v:0 -show_entries stream=bit_rate -of default=noprint_wrappers=1:nokey=1 silent.mp4) && \
    ffmpeg -i silent_movie.mp4 -vf ass=temp.ass -b:v $BITRATE -c:a copy subtitles_burned_in.mp4
    ```

10.  Finally, convert the mp4 into a gif
    
   ```bash
   ffmpeg \
   -i subtitles_burned_in.mp4 \
   -filter_complex "[0:v] fps=12,scale=480:-1,split [a][b];[a] palettegen [p];[b][p] paletteuse" awesome.gif
   ```
---
## And presto!  You now have your own awesome GIF!
