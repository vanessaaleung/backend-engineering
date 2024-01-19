# The Full Stack
## Week 4
- [The Full Stack](#the-full-stack)
  - [Week 4](#week-4)
    - [Web server environments](#web-server-environments)
      - [Deployment: upload data and code to the server](#deployment-upload-data-and-code-to-the-server)
      - [CI/CD](#cicd)
      - [Hosting](#hosting)
      - [Hypervisor](#hypervisor)
      - [Virtual Machines](#virtual-machines)
      - [Containerization](#containerization)
      - [Pod](#pod)
      - [Node](#node)
      - [Cluster](#cluster)
      - [Orchestration](#orchestration)
    - [Cloud Computing](#cloud-computing)
      - [Computing Units](#computing-units)
      - [Storage](#storage)
      - [Databases](#databases)
      - [DNS Servers](#dns-servers)
      - [Bandwidth](#bandwidth)
      - [Uplink and Downlink](#uplink-and-downlink)
    - [Scaling](#scaling)
      - [Load Balancing](#load-balancing)
      - [Sharding](#sharding)
      - [Master-slave replication](#master-slave-replication)
    - [CDN](#cdn)
      - [Advantages](#advantages)
      - [Disadvantages](#disadvantages)
      - [Push CDN](#push-cdn)
      - [Pull CDN](#pull-cdn)

-------

### Web server environments
#### Deployment: upload data and code to the server
  - Developer makes changes to the code
  - Build tool creates Build environment
  - Run test
  - Build tool uploads the code
  - Integration
    - CI: the testing process
    - CD: the uploading process
#### CI/CD
  - Code
  - Version control
  - Build
  - Test Deploy
#### Hosting
  - Server
    - Dedicated: for only your use
    - Virtual servers
  - Serverless: no physical servers
    - No need to configure anything. Managed by the serverless provider: builds code + runs test + deploys code + provides access
#### Hypervisor
  - Virtual servers like individual computers, have dedicated IP addresses to communicate with each other
  - Type-1 hypervisor: works directly with software hardware. application runs faster. e.g. KVM
  - Type-2: works on top of OS, slower than type 1. Simpler management, don't have to deal with hardware. e.g. VirtualBox
#### Virtual Machines
  - Dedicated: servers get dedicated resources (e.g. storage, RAM, cores). Lead to wastage if idle.
  - Shared: can use resources from another server when needed.
#### Containerization
  - **Pack applications and dependencies into containers files.** 
  - Can distribute application into multiple containers.
  - Should be kept clean.
  - Smaller, faster than VMs.
  - e.g. Docker
#### Pod
  - coupled containers
  - share a resource
  - work together
#### Node
  - physical/virtual machine that runs 1+ pods
#### Cluster
  - multiple pods/nodes
  - related/independent
#### Orchestration
  - Managment
  - Deployment
  - Networking
  - Scaling
  - of containers
  - e.g. K8s

-------

### Cloud Computing
- Public cloud: pay-as-you-go, accessible to everyone
  - Scalability, On-demand, Availability
- Private cloud
  - Privately hosted
  - Limited access
  - Better security
  - Costly
- Hybrid cloud
  - Partly hosted on public and partly on private
  - Less expensive
  - Complex

#### Computing Units
- VMs for on-demand deployments
- Host applications
- Can be resized
- 1. Cores
- 2. Memories
- 3. GPUs
- 4. OS
- Volatile storage. Everything gone when reboot.

#### Storage
- Redundant storage, auto backup
- Object storage
  - API
  - Upload/Download
  - Unique URL for file

#### Databases
- In-memory solutions
  - Caching data, faster

#### DNS Servers
- Store domain names against their IP addresses

#### Bandwidth
- Data that comes in/goes out of the srever
  
#### Uplink and Downlink
- Uplink: protocol used to send data outside the server
- Downlink: protocl used to accept incoming data

-------

### Scaling
- Resizing production infra
- Sudden/linear growth
- Vertical
  - Add more resources like CPU, memory, etc.
  - Quicker, easier config
  - cannot be infinite, costly overtime
- Horizontal
  - cost effective
  - adds/removes nodes
  - infinitely scalable
  - takes time
- Auto scaling

#### Load Balancing
- Evenly distributed traffic on multiple nodes (round-robin or based on node health)
- Reverse proxy
  - serve static files (HTML, CSS, etc.) and forward URLs and APIs to the server
  - Improve performance by reducing requests the server handles
- Offload static files
  - to external content network CDN
  - create multiple copies of a static file and serve from the client's nearest data center

#### Sharding
- Database scaling
- Split into multiple chunks
- One server hosts one chunk
- One extra database to keep track of lookup

#### Master-slave replication
- Database scaling
- Replicated on multiple servers
- One master server: where modification happens
- Master server writes changes to slave servers

### CDN
- Store static content files. Deliver content from servers but always from the nearest server to the client.
- A CDN has multiple servers (PoP, point of service)
- Each PoP store a copy of the static files
- If a visitor browses application from Asia, PoP closest to the client's ISP serves the content.

#### Advantages
- Reduced delivery time.
- Reduced workload (# of requests) and bandwidth on the web server.
- Auto-reduces image size.
- Webp: lossless format that preserves the image quality

#### Disadvantages
- Changes may not be instantly visible
- Bandwidth cost

#### Push CDN
- Upload/push file everytime changes are made
- Extra effort

#### Pull CDN
- Updates files automatically
- Auto-detect if the file has changed. If so, pulls the changed files and serves them.
- Used most often.