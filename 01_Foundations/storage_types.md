# Storage Types: Block vs File vs Object

## The Simple Explanation

There are three main ways to store data, each suited for different use cases:

```
┌─────────────────────────────────────────────────────────────────┐
│                    STORAGE TYPES                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   BLOCK STORAGE          FILE STORAGE          OBJECT STORAGE   │
│   ─────────────          ────────────          ──────────────   │
│                                                                 │
│   ┌─┬─┬─┬─┬─┬─┐        /home/                   ┌──────────┐   │
│   │ │ │ │ │ │ │        ├── docs/                │  Object  │   │
│   └─┴─┴─┴─┴─┴─┘        │   ├── a.txt            │  + data  │   │
│   Raw blocks           │   └── b.pdf            │  + meta  │   │
│                        └── pics/                └──────────┘   │
│                            └── cat.jpg                          │
│                                                                 │
│   Like a raw           Like folders            Like a bucket    │
│   hard drive           on your computer        of labeled items │
│                                                                 │
│   Use: Databases       Use: Shared files       Use: Photos,     │
│        VMs                  NAS                     backups     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Block Storage

### What is Block Storage?

Block storage divides data into fixed-size blocks, each with a unique address. It's the lowest-level storage—like raw disk access.

```
┌─────────────────────────────────────────────────────────────────┐
│                      BLOCK STORAGE                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   Data is split into fixed-size blocks:                         │
│                                                                 │
│   Original file: "Hello, this is my document content..."        │
│                                                                 │
│   ┌────────────┐ ┌────────────┐ ┌────────────┐ ┌────────────┐  │
│   │  Block 1   │ │  Block 2   │ │  Block 3   │ │  Block 4   │  │
│   │  "Hello,   │ │  "this is  │ │  "my docu- │ │  "ment..." │  │
│   │  Address:0 │ │  Address:1 │ │  Address:2 │ │  Address:3 │  │
│   └────────────┘ └────────────┘ └────────────┘ └────────────┘  │
│                                                                 │
│   • No file system, no hierarchy                                │
│   • Just raw blocks with addresses                              │
│   • Operating system adds file system on top                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Characteristics

```
┌────────────────────────────────────────────────────────────────┐
│                BLOCK STORAGE CHARACTERISTICS                   │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  ✓ HIGHEST PERFORMANCE                                         │
│    └── Direct disk access                                      │
│    └── Lowest latency                                          │
│    └── Best IOPS                                               │
│                                                                │
│  ✓ FLEXIBLE                                                    │
│    └── Format with any file system (ext4, NTFS, XFS)           │
│    └── Use for any application                                 │
│                                                                │
│  ✓ GREAT FOR DATABASES                                         │
│    └── Databases need raw, fast storage                        │
│    └── Support for atomic operations                           │
│                                                                │
│  ✗ NOT SHAREABLE (easily)                                      │
│    └── Typically attached to one server                        │
│    └── Multi-attach is complex                                 │
│                                                                │
│  ✗ NO BUILT-IN METADATA                                        │
│    └── Just blocks, no file names or attributes                │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

### Use Cases

| Use Case | Why Block Storage? |
|----------|-------------------|
| Databases (MySQL, PostgreSQL) | Need raw performance, IOPS |
| Virtual Machines | Boot from block device |
| Transactional Systems | Low latency, atomic writes |
| High-performance computing | Maximum throughput |

### Examples

- **AWS:** EBS (Elastic Block Store)
- **GCP:** Persistent Disk
- **Azure:** Managed Disks
- **Physical:** SAN (Storage Area Network), local SSD

---

## File Storage

### What is File Storage?

File storage organizes data in a hierarchical structure of files and directories—like your computer's file system.

```
┌─────────────────────────────────────────────────────────────────┐
│                       FILE STORAGE                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   Hierarchical structure:                                       │
│                                                                 │
│   /                                                             │
│   ├── home/                                                     │
│   │   ├── alice/                                                │
│   │   │   ├── documents/                                        │
│   │   │   │   ├── report.docx                                   │
│   │   │   │   └── notes.txt                                     │
│   │   │   └── pictures/                                         │
│   │   │       └── vacation.jpg                                  │
│   │   └── bob/                                                  │
│   │       └── work/                                             │
│   │           └── project.pdf                                   │
│   └── shared/                                                   │
│       └── team-docs/                                            │
│           └── handbook.pdf                                      │
│                                                                 │
│   Access via path: /home/alice/documents/report.docx            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Characteristics

```
┌────────────────────────────────────────────────────────────────┐
│                FILE STORAGE CHARACTERISTICS                    │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  ✓ FAMILIAR INTERFACE                                          │
│    └── Works like your computer's file system                  │
│    └── Paths, directories, files                               │
│                                                                │
│  ✓ EASY SHARING                                                │
│    └── Multiple servers can mount same file system             │
│    └── NFS, SMB/CIFS protocols                                 │
│                                                                │
│  ✓ FILE-LEVEL OPERATIONS                                       │
│    └── Edit part of a file without rewriting                   │
│    └── File locking                                            │
│                                                                │
│  ✗ SCALABILITY LIMITS                                          │
│    └── Hierarchy doesn't scale to billions of files            │
│    └── Directory lookups become slow                           │
│                                                                │
│  ✗ LOWER PERFORMANCE than block                                │
│    └── File system overhead                                    │
│    └── Metadata operations                                     │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

### Use Cases

| Use Case | Why File Storage? |
|----------|-------------------|
| Shared files | Multiple users/servers need access |
| Home directories | User files on network |
| Content management | Documents, media files |
| Legacy applications | Apps expecting file system interface |

### Examples

- **AWS:** EFS (Elastic File System)
- **GCP:** Filestore
- **Azure:** Azure Files
- **Protocols:** NFS, SMB/CIFS
- **Physical:** NAS (Network Attached Storage)

---

## Object Storage

### What is Object Storage?

Object storage stores data as objects in a flat namespace. Each object contains data, metadata, and a unique identifier.

```
┌─────────────────────────────────────────────────────────────────┐
│                      OBJECT STORAGE                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   Flat structure with unique keys:                              │
│                                                                 │
│   ┌───────────────────────────────────────────────────────┐    │
│   │                     BUCKET                            │    │
│   │                                                       │    │
│   │  ┌─────────────────────────────────────────────────┐ │    │
│   │  │ Key: "users/alice/profile-pic.jpg"              │ │    │
│   │  │ Data: [binary image data]                       │ │    │
│   │  │ Metadata:                                       │ │    │
│   │  │   - content-type: image/jpeg                    │ │    │
│   │  │   - size: 2.4MB                                 │ │    │
│   │  │   - created: 2024-01-15                         │ │    │
│   │  │   - custom-tag: vacation                        │ │    │
│   │  └─────────────────────────────────────────────────┘ │    │
│   │                                                       │    │
│   │  ┌─────────────────────────────────────────────────┐ │    │
│   │  │ Key: "backups/db-2024-01-15.sql.gz"            │ │    │
│   │  │ Data: [compressed SQL dump]                     │ │    │
│   │  │ Metadata:                                       │ │    │
│   │  │   - content-type: application/gzip             │ │    │
│   │  │   - size: 500MB                                 │ │    │
│   │  │   - storage-class: GLACIER                      │ │    │
│   │  └─────────────────────────────────────────────────┘ │    │
│   │                                                       │    │
│   └───────────────────────────────────────────────────────┘    │
│                                                                 │
│   Access via: GET bucket-name/users/alice/profile-pic.jpg       │
│   (The "/" is just part of the key, not a real folder!)         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Characteristics

```
┌────────────────────────────────────────────────────────────────┐
│               OBJECT STORAGE CHARACTERISTICS                   │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  ✓ MASSIVE SCALABILITY                                         │
│    └── Billions of objects, petabytes of data                  │
│    └── Flat namespace = no directory bottleneck                │
│                                                                │
│  ✓ RICH METADATA                                               │
│    └── Custom metadata per object                              │
│    └── Searchable attributes                                   │
│                                                                │
│  ✓ HTTP/REST API ACCESS                                        │
│    └── Access from anywhere via HTTP                           │
│    └── Easy integration with web applications                  │
│                                                                │
│  ✓ COST EFFECTIVE                                              │
│    └── Cheaper than block/file at scale                        │
│    └── Storage tiers (hot, warm, cold, archive)                │
│                                                                │
│  ✓ BUILT-IN DURABILITY                                         │
│    └── Data replicated across locations                        │
│    └── 99.999999999% (11 nines) durability                     │
│                                                                │
│  ✗ NO PARTIAL UPDATES                                          │
│    └── Must replace entire object                              │
│    └── Not suitable for frequently modified files              │
│                                                                │
│  ✗ EVENTUAL CONSISTENCY (traditionally)                        │
│    └── May take time for changes to propagate                  │
│    └── (S3 now offers strong consistency)                      │
│                                                                │
│  ✗ HIGHER LATENCY                                              │
│    └── Not suitable for low-latency requirements               │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

### Use Cases

| Use Case | Why Object Storage? |
|----------|-------------------|
| Static website hosting | Images, CSS, JS files |
| Backup and archive | Cheap, durable, scalable |
| Data lakes | Store raw data for analytics |
| Media storage | Video, audio, images |
| Log storage | Application and system logs |
| Machine learning datasets | Large datasets |

### Examples

- **AWS:** S3 (Simple Storage Service)
- **GCP:** Cloud Storage
- **Azure:** Blob Storage
- **Open Source:** MinIO, Ceph

---

## Comparison

```
┌─────────────────────────────────────────────────────────────────┐
│                    STORAGE COMPARISON                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   Feature        │ Block      │ File       │ Object             │
│   ───────────────┼────────────┼────────────┼──────────────────  │
│   Performance    │ Highest    │ Medium     │ Lower              │
│   Scalability    │ Limited    │ Medium     │ Virtually infinite │
│   Sharing        │ Hard       │ Easy       │ Via HTTP           │
│   Cost           │ High       │ Medium     │ Low                │
│   Metadata       │ None       │ Basic      │ Rich, custom       │
│   Access         │ Device     │ File path  │ HTTP/REST API      │
│   Modify in place│ Yes        │ Yes        │ No (replace only)  │
│   Protocols      │ iSCSI, FC  │ NFS, SMB   │ HTTP/REST          │
│   Best for       │ Databases  │ File share │ Unstructured data  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Visual Comparison

```
┌─────────────────────────────────────────────────────────────────┐
│                    WHEN TO USE EACH                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   Performance                                                   │
│        ▲                                                        │
│        │   ┌─────────┐                                         │
│   High │   │  Block  │                                         │
│        │   └─────────┘                                         │
│        │        ┌─────────┐                                     │
│   Med  │        │  File   │                                     │
│        │        └─────────┘                                     │
│        │              ┌─────────┐                               │
│   Low  │              │ Object  │                               │
│        │              └─────────┘                               │
│        └────────────────────────────────────────────► Scale     │
│              Low         Med         High                       │
│                                                                 │
│                                                                 │
│   DECISION TREE:                                                │
│   ──────────────                                                │
│                                                                 │
│   Need raw disk for DB/VM?                                      │
│        └── YES → Block Storage                                  │
│        └── NO ↓                                                 │
│                                                                 │
│   Need shared file access (NFS/SMB)?                            │
│        └── YES → File Storage                                   │
│        └── NO ↓                                                 │
│                                                                 │
│   Storing unstructured data (images, videos, backups)?          │
│        └── YES → Object Storage                                 │
│        └── NO → Evaluate specific needs                         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Real-World Architecture Example

```
┌─────────────────────────────────────────────────────────────────┐
│              E-COMMERCE STORAGE ARCHITECTURE                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   ┌───────────────────────────────────────────────────────┐    │
│   │                    APPLICATION                        │    │
│   └─────────┬───────────────┬──────────────────┬──────────┘    │
│             │               │                  │               │
│             ▼               ▼                  ▼               │
│   ┌─────────────────┐ ┌─────────────┐ ┌────────────────────┐   │
│   │  BLOCK STORAGE  │ │FILE STORAGE │ │  OBJECT STORAGE    │   │
│   │  (EBS/PD)       │ │(EFS/NFS)    │ │  (S3)              │   │
│   ├─────────────────┤ ├─────────────┤ ├────────────────────┤   │
│   │                 │ │             │ │                    │   │
│   │ • PostgreSQL    │ │ • Shared    │ │ • Product images   │   │
│   │   database      │ │   config    │ │ • User uploads     │   │
│   │ • Redis data    │ │   files     │ │ • Backups          │   │
│   │ • Application   │ │ • Log files │ │ • Static website   │   │
│   │   boot disk     │ │ • ML models │ │   assets           │   │
│   │                 │ │             │ │ • Analytics data   │   │
│   │ Fast, low-level │ │ Shared      │ │ Scalable, cheap    │   │
│   │ access needed   │ │ access      │ │ archival           │   │
│   │                 │ │ needed      │ │                    │   │
│   └─────────────────┘ └─────────────┘ └────────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Interview Questions

### Question 1: "When would you use object storage vs. block storage?"

**Strong Answer:**
```
OBJECT STORAGE when:
• Storing unstructured data (images, videos, backups)
• Need massive scale (petabytes)
• Data is write-once, read-many
• Need HTTP/REST API access
• Cost is a concern for large data volumes
• Don't need to modify files in place

Examples: User uploads, backups, data lakes, static assets

BLOCK STORAGE when:
• Running databases (MySQL, PostgreSQL)
• Need low latency, high IOPS
• Need to modify data in place
• Running VMs that need boot disks
• Application needs raw disk access

Examples: Database storage, VM disks, high-performance computing

KEY INSIGHT: Use both in the same system!
• Database data → Block storage
• User uploads → Object storage
• Application code → Block storage (boot disk)
• Backups → Object storage (cheap, durable)
```

### Question 2: "How would you design storage for a photo-sharing app?"

**Strong Answer:**
```
STORAGE ARCHITECTURE:

1. USER-UPLOADED PHOTOS → Object Storage (S3)
   • Massive scale (billions of photos)
   • Cheap storage
   • CDN integration for fast delivery
   • Storage classes: frequently accessed (hot) → archive

2. METADATA DATABASE → Block Storage (RDS on EBS)
   • User info, photo metadata
   • Needs fast queries, indexing
   • ACID transactions

3. THUMBNAILS → Object Storage (S3) + CDN
   • Generated versions of photos
   • Serve via CDN for low latency

4. SEARCH INDEX → Block Storage (Elasticsearch on EBS)
   • Fast searches require local SSD

FLOW:
1. User uploads photo
2. Store original in S3 (object storage)
3. Generate thumbnails, store in S3
4. Store metadata in PostgreSQL (block storage)
5. Index for search in Elasticsearch (block storage)
6. Serve via CDN (pulling from S3)
```

---

## Key Takeaways

```
┌────────────────────────────────────────────────────────────────┐
│                    REMEMBER THIS                               │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  1. BLOCK STORAGE                                              │
│     • Raw disk access, highest performance                     │
│     • Use for: databases, VMs                                  │
│     • Examples: AWS EBS, local SSD                             │
│                                                                │
│  2. FILE STORAGE                                               │
│     • Hierarchical files/folders, shareable                    │
│     • Use for: shared files, NAS                               │
│     • Examples: AWS EFS, NFS                                   │
│                                                                │
│  3. OBJECT STORAGE                                             │
│     • Flat namespace, massive scale, cheap                     │
│     • Use for: images, backups, data lakes                     │
│     • Examples: AWS S3, GCS                                    │
│                                                                │
│  4. Most systems use MULTIPLE storage types                    │
│     • Databases on block                                       │
│     • User files on object                                     │
│     • Shared config on file                                    │
│                                                                │
│  5. Object storage can't be modified in place                  │
│     Block storage offers the best performance                  │
│     File storage is best for sharing                           │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

---

## What's Next?

Now that you understand storage types, let's dive into the most important storage decision in system design: SQL vs NoSQL databases.

**Next:** [SQL vs NoSQL →](sql_vs_nosql.md)
