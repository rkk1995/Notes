# CDN

A content delivery network (CDN) is a globally distributed network of proxy servers, serving content from locations closer to the user. Generally, static files such as HTML/CSS/JS, photos, and videos are served from CDN, although some CDNs such as Amazon's CloudFront support dynamic content. The site's DNS resolution will tell clients which server to contact.

Serving content from CDNs can significantly improve performance in two ways:

- Users receive content from data centers close to them
  - Reduce latency and round-trip time
- Ensure sufficient bandwidth for high traffic periods
- They reduce the workload on origin servers
  - Since it takes a lot of computing power for a single server to respond to requests, and even more so for video live streaming, CDNs essentially protect the origin servers from overload and keep it operational.

## Push CDNs

Push CDNs receive new content whenever changes occur on your server. You take full responsibility for providing content, uploading directly to the CDN and rewriting URLs to point to the CDN. You can configure when content expires and when it is updated. Content is uploaded only when it is new or changed, minimizing traffic, but maximizing storage.

Sites with a small amount of traffic or sites with content that isn't often updated work well with push CDNs. Content is placed on the CDNs once, instead of being re-pulled at regular intervals.

## Pull CDNs

Pull CDNs grab new content from your server when the first user requests the content. You leave the content on your server and rewrite URLs to point to the CDN. This results in a slower request until the content is cached on the CDN.

A [time-to-live (TTL)](https://en.wikipedia.org/wiki/Time_to_live) determines how long content is cached. Pull CDNs minimize storage space on the CDN, but can create redundant traffic if files expire and are pulled before they have actually changed.

Sites with heavy traffic work well with pull CDNs, as traffic is spread out more evenly with only recently-requested content remaining on the CDN.

## Disadvantages

- CDN costs could be significant depending on traffic, although this should be weighed with additional costs you would incur not using a CDN.
- Content might be stale if it is updated before the TTL expires it.
- CDNs require changing URLs for static content to point to the CDN.

## Live Streaming CDN

How does a live streaming CDN work?

- Step 1: Video capture
  - Content creator captures the raw data or visual information using a camera. The data is represented in binary 1s and 0s in the device.
- Step 2: Segmentation
  - Video file is broken down into smaller parts of a few seconds in length. Breaking them down into segments helps in streaming the entire video bit by bit.
- Step 3: Compression and encoding
  - Each of the segments are compressed and encoded. Compressing removes redundant visual information such as a background that does not change in the video. This makes it easy to render just the moving frames in the video before streaming. Encoding is a process that is necessary to convert the data into a format that is compatible with the variety of devices that the end user consumes the content on. For example, H.264, HEVC, VP9 and AV1 are some of the popular formats that videos are encoding into.
- Step 4: Content Distribution and CDN Caching
  - Next, the segmented, compressed and encoded video is distributed to end users. When the end user accesses a website or plays a video, their device (client) sends a request to the origin server to retrieve these files. Now if the users are located in close proximity to the server, or within a nearby region, this should not be a problem and the video files are streamed without much of an issue.
  - In fact, if your viewership is small and they are not widely distributed, the single server can stream to all your users. There is no need to introduce more elements into your streaming workflow.
  - But when the users are dispersed across a larger geographical area, in some cases across different countries, the round-trip-time for the server to deliver the content can be longer, resulting in delays or latency. This results in a below par user experience and one that is inconsistent across all of the video's consumers.
  - Using a CDN solves this problem by [caching the content](https://www.cdnetworks.com/knowledge-center/what_is_cache_control/) in its distributed network of streaming servers. The CDN server closest to a particular end user will take care of delivering the content to that user.
- Step 5: Decoding and playback
  - Once the video data reaches the users, their devices will decode and decompress the video segment by segment into the binary raw data. And with a video player, the user is able to see the visual information and play the video.

Advantages

- Ensure sufficient bandwidth for high traffic periods
- Reduce latency and round-trip time
- Help in live streaming to a global audience
- Reduce the workload on origin servers
