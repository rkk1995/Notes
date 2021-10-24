# Back of Envelope Calculations

*Note: All the numbers in this post are heavily rounded as their purpose is to give a rough guide for design decisions in the moment. You should always do more precise calculations before starting on a project/feature.*

## Byte Number Sizes

The number of zeros after thousands increments by 3.

- 1 Byte = 8 Bits
- 1 KB = $10^3$ Bytes
- 1 MB = $10^6$ Bytes
- 1 GB = $10^9$ Bytes
- 1 TB = $10^{12}$ Bytes
- 1 PB = $10^{15}$ Bytes
  
### Useful Calculations

`x Million users * y KB = xy GB`

example: 1M users* a documents of 100KB per day = 100GB per day.

`x Million users * y MB = xy TB`

example: 200M users* a short video of 2MB per day = 400TB per day.

## Object Sizes

### Data

The numbers vary depending on the language and implementation.

- char: 1B (8 bits)
- char (Unicode): 2B (16 bits)
- Short: 2B (16 bits)
- Int: 4B (32 bits)
- Long: 8B (64 bits)
- UUID/GUID: 16B

### Objects

- File: 100 KB
- Web Page: 100 KB (not including images)
- Picture: 200 KB
- Short Posted Video: 2MB
- Steaming Video: 50MB per minute
- Long/Lat: 8B

### Lengths

- Maximum URL Size: ~2000 (depends on browser)
- ASCII charset: 128
- Unicode charset: 143, 859

## Usage

### Users

- Facebook: 2.27B | YouTube: 2B | Instagram: 1B
- Pinterest: 332M | Twitter: 330M | Onedrive: 250M
- TikTok: 3.7M

### Visits

- Facebook: 26.12B | Twitter: 6.34B | Pinterest: 1.32B
- Spotify: 293M | Ikea: 233M | Nike: 110M
- Argos: 54M | John Lewis: 37M |Superdry: 3.5M
- Virgin Money: 1.8M | Aviva: 1.61M

### Cost of Operations

- Read sequentially from HDD: 30 MB/s
- Read sequentially from SSD: 1 GB/s
- Read sequentially from memory: 4 GB/s
- Read sequentially from 1Gbps Ethernet: 100MB/s
- Cross continental network: 6--7 world-wide round trips per second.

## Systems

These are not exact numbers, which very much depend on the implementation and what is hosting the service. The purpose of the numbers is to have a general idea of the performance across different types of services.

### SQL Databases

- Storage: 60TB
- Connections: 30K
- Requests: 25K per second

### Cache

[[Redis --- Requests](https://redis.io/topics/benchmarks)][[Redis --- connections](https://redis.io/topics/clients#:~:text=In%20Redis%202.4%20there%20was,conf.)]

- Storage: 300 GB
- Connections: 10k
- Requests: 100k per second

### Web Servers

- Requests: 5--10k requests per second

### Queues/Streams

[[Pub/Sub --- limits](https://cloud.google.com/pubsub/quotas)][[Kinesis --- limits](https://docs.aws.amazon.com/streams/latest/dev/service-sizes-and-limits.html)][[SQS --- limits](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/quotas-messages.html)]

- Requests: 1000--3000 requests/s
- Throughput: 1MB-50MB/s (Write) / 2MB-100MB/s (Read)

### Scrapers

--------

[[Colly --- go scraper](https://github.com/gocolly/colly)]

- Requests: 1000 per second
