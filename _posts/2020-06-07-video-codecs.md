---
title:  "Video codecs?"
category: knowledge_base
tag: [WebRTC]
date: 2020-06-07
---

## Video Codecs

What are codecs? 

**Video codecs** are implementation of video coding formats, and provide encoding and decoding for digital video, often including the use of video compression/decompression.

**Video coding formats** are how digital video files are optimized to be delivered to different platforms, programs, and devices. It includes information such as how the video is stored, transmitted, and viewed. 

**Video Containers** are the container of the codecs, combining video codec, audio codec, and other metadata into a single container that can be played altogether. Video containers support different types of video codecs, so only certain types of containers may be compatible with certain types of video/audio codecs. 

Use of different codecs is very critical for services that include video usage as it can determine the efficiency and availability of video contents. Video codecs determine the quality, size, and the compatibility of the video. 

### Video Encoding

To be available on many different devices, raw video has to be converted into digital format that's compatible with other devices. Video encoding involves two main processes: compression and transcoding. 

- Compression determines how the size of video file is managed. By decreasing the size of the raw video file, they can be stored more easily and transferred between devices more easily as well. 
- Transcoding includes the total audio and video conversion process from one video format to another. It allows video files to be converted and be compatible with different video players/devices. 

### Video Codec Qualities

There are few different components that should be considered when comparing video codecs. 
- **Video quality per bitrate**: Either subjective or objective video quality can be considered.
- **Performance characteristics**: Compression/decompression speed, supported profiles, resolutions, rate controls, etc
- **General software characteristics**

Video quality depends heavily on the compression format that the codec uses. Compression specifications define how raw videos are compressed, such as with simple bit compression, psycho-visual and motion sumarization. It also determines how the output is stored as a bit stream. Decoder component of a codec determines how the stored files are recognized and how they are interpreted to be displayed. 

Encoding can be done with either *lossy* compression or *lossless* compression. Lossy compression attempts to keep the video running smooth by dropping nonessential pixels and simplifying the video. This can result in a fuzzy video, but the frame rate would be better than lossless compression, which preserves the quality of the original file by copying each pixel of the data. 

Lossy compression offers smaller file size, lower video quality, and better speed; and lossless compression offers larger file size, better video quality, and lower speed. 

#### Components that affect video quality/size
- Color depth: Higher color depth results in higher quality of color in the video. However, higher color dephs may result in larger compressed file sizes. The internal storage format and how the colors are saved determine the size of the compressed data.
- Frame rate: Frame rate affects the smoothness of the video. Because higher frame rate contains more frames per second, it results in a larger compressed video files. 
- Motion: Video compression is usually done by comparing multiple frames, finding the pixels where they differ, and constructing a intermediate frame that is upddated from the previous frame to approximate the appearance of the following frames. Because larger amount of motion results in less amount of different frames, the video file becomes less-compressed.
- Noise: Noise in videos creates variability in the frames, making the compression more difficult, and reducing the quality of compressed video. 

#### Result of compression on the video quality/size
- Lossless compression: Quality remains same as the original video, but reuslting size would be too large for general usage. 
- Lossy compression: Quality is reduced to some degree in order to compress the video efficiently. Resulting size of the video is smaller, and can be used more broadly.
- Bit rate: Higher bit rate results in higher quality, but also leads to larger output files. 


### Few Types of codecs

##### H.264
H.264, also reffered as Advanced Video Coding(AVC), is one of the most popular encoding format today. It's based on block-oriented, motion-compensated integer-DCT coding. 

H.264 can be packaged into various video containers, such as .mp4, .mov, .ts, .F4v. Thus, it can be played on virtually any device, delivering a seamless video quality. 

It uses very little storage compared to other codecs, and requires very little network bandwidth which makes it suitable for streaming services. 

##### VP8
VP8 is a video compression format which is extended from VP7. It is open source alternative to H.264.

Because it is designed for newer mobile devices, it offers high compression efficiency and low computational complexity. 

Specifically, it has low bandwidth, wide range of client devices and web video formats. VP8 uses hybrid transform with adaptive quantization based on DCT and WHT. It performs equally or better than H.264. 

##### VP9 
VP9 is a successor to VP8.

It is designed to use more CPU resources to compress video, but uses less bandwith and delivers clearer video compared to VP8. It aims to reduce the bit rate by 50% compared to VP8 while remaining the same video quality, and delivering better compression efficiency. 

VP9 is custommized for high-resolution videos greater than 1080p and enables *lossless compression*. 

###### Video Quality Glossary
- Bitrate: rate at which bits are transferred. Measures how much data is transmitted in a given amount of time. 
- FPS(Frame rate Per Second): frequency at which frames are rendered on a display. Higher FPS results in a smoother video.
- Resolution: number of distinct pixels that could be displayed on a display. Higher resolution results in a clearer video. 

Further study: https://developer.mozilla.org/en-US/docs/Web/Media/Formats/Video_codecs