# Live Streaming Architecture with AWS MediaConvert & CloudFront

VideoJS by default uses standard HTML5 `<video>` with HTTP progressive download when you point it directly at an S3 MP4 URL.

## Current Setup Limitations

Progressive streaming has these issues:

- **No quality adaptation** - same bitrate for all network conditions
- **Wastes bandwidth** - downloads entire video even if user stops watching
- **Slow startup** - must download enough data before playback
- **No resolution switching** - user can't choose quality
- **Poor mobile experience** - same quality for WiFi and cellular

## Adaptive Streaming is the Solution

You need **HLS (HTTP Live Streaming)** or **DASH** - these break videos into small chunks and allow quality switching based on network conditions.

### Implementation: AWS MediaConvert + CloudFront

**Architecture Overview:**

```
Upload MP4 → MediaConvert → HLS files (.m3u8 + .ts segments) → S3 → CloudFront → React App
```

## Step 1: Create S3 Buckets

```
# Source videos bucket
source-videos-bucket

# Processed videos bucket (HLS output)
streaming-videos-bucket
```

## Step 2: Set up AWS MediaConvert

**Input:** Your source MP4 from S3

**Output groups:**
- Apple HLS
- Multiple renditions (240p, 360p, 480p, 720p, 1080p)

```json
{
  "OutputGroups": [
    {
      "Name": "Apple HLS",
      "OutputGroupSettings": {
        "Type": "HLS_GROUP_SETTINGS",
        "HlsGroupSettings": {
          "SegmentLength": 6,
          "MinSegmentLength": 0,
          "Destination": "s3://streaming-videos-bucket/hls/"
        }
      },
      "Outputs": [
        {
          "NameModifier": "_240p",
          "VideoDescription": {
            "Width": 426,
            "Height": 240,
            "CodecSettings": {
              "Codec": "H_264",
              "H264Settings": {
                "Bitrate": 400000
              }
            }
          }
        },
        {
          "NameModifier": "_360p",
          "VideoDescription": {
            "Width": 640,
            "Height": 360,
            "CodecSettings": {
              "Codec": "H_264",
              "H264Settings": {
                "Bitrate": 800000
              }
            }
          }
        },
        {
          "NameModifier": "_480p",
          "VideoDescription": {
            "Width": 854,
            "Height": 480,
            "CodecSettings": {
              "Codec": "H_264",
              "H264Settings": {
                "Bitrate": 1400000
              }
            }
          }
        },
        {
          "NameModifier": "_720p",
          "VideoDescription": {
            "Width": 1280,
            "Height": 720,
            "CodecSettings": {
              "Codec": "H_264",
              "H264Settings": {
                "Bitrate": 2800000
              }
            }
          }
        },
        {
          "NameModifier": "_1080p",
          "VideoDescription": {
            "Width": 1920,
            "Height": 1080,
            "CodecSettings": {
              "Codec": "H_264",
              "H264Settings": {
                "Bitrate": 5000000
              }
            }
          }
        }
      ]
    }
  ]
}
```

## Step 3: Automate with Lambda

Trigger MediaConvert when new videos are uploaded:

```javascript
// Lambda function triggered by S3 upload
exports.handler = async (event) => {
  const AWS = require('aws-sdk');
  const mediaconvert = new AWS.MediaConvert({ endpoint: 'YOUR_ENDPOINT' });
  
  const s3Record = event.Records[0].s3;
  const bucket = s3Record.bucket.name;
  const key = decodeURIComponent(s3Record.object.key.replace(/\+/g, ' '));
  
  const params = {
    Role: 'YOUR_MEDIACONVERT_ROLE_ARN',
    Settings: {
      Inputs: [{
        FileInput: `s3://${bucket}/${key}`
      }],
      OutputGroups: [ /* Your HLS settings */ ]
    }
  };
  
  await mediaconvert.createJob(params).promise();
};
```

### Alternative Approach: TypeScript with Kafka

Instead of Lambda, consider using a **TypeScript-based Bun Docker container** that subscribes to Kafka while continuing to use AWS MediaConvert.

**Key Features:**
- Security scanning to ensure no viruses
- Integration with AWS Cloud Storage Security
- Control flow monitoring to track asset processing status
- Catalog updates when assets are ready for live deployment

**System Control & Orchestration:**

The system needs to answer critical questions:
- If trailer 1 is good but trailer 2 has issues, how do we alert the publisher?
- Should the publisher push live without the new trailer or hold the release?
- When should the publisher be updated?
- When should alerts be triggered?

**Architecture Benefits:**
- Multiple orchestrators for workflow management
- End-to-end (E2E) system visibility
- Clear identification of managed services and their integration points
- Cost tracking for each service and running jobs

## Step 4: Create CloudFront Distribution

**Origin:** Your streaming-videos-bucket

**Enable CORS**

**Cache behaviors optimized for video:**
- `.m3u8` files: Low TTL (5-10 seconds)
- `.ts` segments: High TTL (1 year)

### CloudFront CORS Policy

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowPublicRead",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::streaming-videos-bucket/*"
    }
  ]
}
```

### S3 CORS Configuration

```json
[
  {
    "AllowedHeaders": ["*"],
    "AllowedMethods": ["GET", "HEAD"],
    "AllowedOrigins": ["*"],
    "ExposeHeaders": ["Content-Length", "Content-Range"]
  }
]
```

## Step 5: React Implementation

```bash
npm install video.js videojs-contrib-quality-levels videojs-hls-quality-selector
```

## Step 6: React Video Player Component Creation and Rendering

See the code file for implementation details.

---

## Advanced Optimizations

### 1. Thumbnail Previews

Generate sprite sheets during MediaConvert:

```json
{
  "Extension": "jpg",
  "NameModifier": "_thumbnail",
  "VideoDescription": {
    "Width": 192,
    "Height": 108
  }
}
```

### 2. Multi CDN Strategy

Use multiple CDNs for failover:

```javascript
const sources = [
  { src: 'https://cdn1.example.com/video.m3u8', type: 'application/x-mpegURL' },
  { src: 'https://cdn2.example.com/video.m3u8', type: 'application/x-mpegURL' }
];
```

### 3. Preload Strategy

```javascript
// Preload next video in playlist
const preloadLink = document.createElement('link');
preloadLink.rel = 'preload';
preloadLink.as = 'fetch';
preloadLink.href = nextVideoUrl;
document.head.appendChild(preloadLink);
```

### 4. Analytics Integration (Optional)

```javascript
player.on('timeupdate', () => {
  // Track watch time, quality changes, buffering events
  analytics.track('video_progress', {
    videoId,
    currentTime: player.currentTime(),
    quality: currentQuality,
    bufferHealth: player.buffered().length
  });
});
```

---

## Cost Optimization Tips

- Use S3 Intelligent-Tiering for infrequently accessed videos
- Enable CloudFront compression for manifest files
- Set appropriate TTLs:
  - `.m3u8` master playlist: 10 seconds
  - `.ts` segments: 1 year (immutable)
- Use AWS Elemental MediaPackage if you need live streaming

---

## Expected Performance Improvements

| Metric | Progressive | HLS Adaptive |
|--------|-------------|--------------|
| Startup time | 3-5s | 1-2s |
| Buffering | Frequent | Rare |
| Bandwidth savings | 0% | 30-50% |
| Quality adaptation | None | Real-time |
| Web experience | Poor | Excellent |

--- 

<button onclick="window.scrollTo({top: 0, behavior: 'instant'})" style="position: fixed; bottom: 30px; right: 30px; z-index: 99; padding: 12px 16px; background-color: #007bff; color: white; border: none; border-radius: 50%; cursor: pointer; font-size: 18px; width: 50px; height: 50px; text-align: center; line-height: 24px; box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2); font-weight: bold; transition: background-color 0.3s;" onmouseover="this.style.backgroundColor='#0056b3'" onmouseout="this.style.backgroundColor='#007bff'" onmousedown="this.style.backgroundColor='#003d82'" onmouseup="this.style.backgroundColor='#0056b3'" title="Go to top">↑</button>
