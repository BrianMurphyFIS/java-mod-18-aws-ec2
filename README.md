# AWS EC2


## Learning Goals

- Understanding VM Instances
- Understanding EC2 Instance Types
- Understanding EC2 Images
- Understanding Regions/Availability Zones
- Understanding Auto-Scaling
- Understanding Instance Access and Bootstrapping


## EC2 VM Instances
        
If you recall from our previous overview of Cloud, Cloud Platforms can be though
of as a Virtual Datacenter. One of the core components of a functioning
Datacenter, whether physical or virtual, is the concept of general purpose
computation servers. Without these, custom developed applications cannot be run.
In the case of AWS, this role is handled mainly by Elastic Compute Cloud(EC2).

EC2 handles the creation and management of Virtual Machines(VMs). Both the core
service of provisioning the underlying machine hardware, as well as all the
ancillary components that allow these machines to operated at scale:
i.e. networking components, scaling features, etc.

If you have ever worked with another featureful Hypervisor platform such as
VMware vSphere, Xen, Hyper-V, or another Cloud Provider's offering, EC2 should
appear quite similar as it offers many of the same capabilities. If you've used
a less featureful local platform such as VirtualBox, VMware Horizon, or KVM,
you've already been introduced to the main concepts behind Hypervisors and
Virtual Machines, but without the more enterprise necessary components for scale
mentioned above.

EC2 falls into the category of Infrastructure as a Service(IaaS), and should be
primarily used for cases and situations which call for the construction of
custom infrastructure(servers, operating systems, networking, etc.). EC2 can
also be a valid platform for any other higher abstracted software components,
but those situations tend to be more easily addressable with other AWS offerings
focused on Platform as a Service(PaaS) or Software as a Service(SaaS).

### What are EC2 Instances?

The basic component of EC2 is the VM Instance. You can think of this in terms of
a normal Virtual Machine, and in many cases it is quite similar to other Virtual
Machine Instances you might have used on other Hypervisor platforms.

With EC2, you are charged for the runtime of any EC2 Instances in your account,
and there is very little limit to the maximum or minimum scale you are able to
achieve. While we are not explicitly covering billing of resources, you should
always keep this fact in mind, and keep an eye on the billing dashboards and
limits. An accidental selection or typo could easily lead to hefty expenses
accruing quickly.

### State and Lifecycle of VM Instances
        
EC2 Instances have operational states and transitions that mimic the states and
lifecycle of traditional physical servers or workstations. As you are managing
these systems over the course of an application's development and operational
lifespan, any used Instance might eventually pass through all these states.
Most of these should be self-explanatory for anyone with even basic computer
experience:

- Start(Pending/Running)

  When you initially create an Instance, it will `Start` and pass to the
  `Running` state. The virtual hardware is powered on, the BIOS boots, and the
  Operating System starts running in this environment. The instance will be
  listed as `Running` once this process is fully complete.
  
- Stop(Stopping/Stopped)

  When you `Stop` a `Running` instance, it will transition to the `Stopped`
  state. `Stopping` the instance can either be done from the running OS, the EC2
  Web Console, or the EC2 CLI.
  
  Note that a VM `Stop` triggered from the OS might potentially hang during the
  shutdown, due to stalling or failing of whatever OS operation is taking place.

- Reboot(Rebooting)

  A `Reboot` is essentially a `Stop` followed by a `Start`. The only real
  differentiating factor with this implementation in EC2 is that instance
  billing is handled differently between the two cases. With a `Reboot` there is
  no interruption of charge, while a `Stop`/`Start` does cause an interruption.
  This might matter greatly depending on the specifics of the Instance Types and
  OS Images, as billing can be handled differently between all the Types and
  Images.
  
- Terminate(Terminated)

  Once an Instance is no longer needed, it can be deleted by triggering a
  `Terminate`. Once triggered, it will be removed from use, and set as
  `Terminated`. It will remain visible in this state for an hour afterwards
  while the EC2 systems schedule cleanup of the resources. It cannot be
  recovered after it has been `Terminated`.
  
  Note that this `Terminated` state might cause issues for automation or CLI
  usage: e.g. duplicate VM names could exist if one is in the `Terminated` state.

- Hibernate

  `Hibernate` in EC2 is a similar concept to the `Hibernate` state common with
  most Operating Systems. The memory of the VM will be written out to disk, and
  then the Instance will be `Stopped`. Once you `Start` the Instance again, the
  memory contents are read in from disk, and the OS continues operation again
  from the same state where it left off.
  
  With `Hibernate` in EC2, you are not charged for the running time of the
  `Stopped` Instance, but you will be billed for the additional Disk space
  needed to store the memory contents.
  
  This state might be useful for infrequently used systems that require long
  start-up times.
  
- Recovery

  Instance `Recovery` is a state that might be seen when underlying EC2
  infrastructure fails. Migration of the Virtual Machine Instance from a failed
  Hypervisor server to another working cluster node will take place, and is
  fully managed on the EC2 system side.
  
  It can still lead to unavoidable downtime during the recovery, as there is no
  native High Availability for individual Instances. Redundancy needs to be
  implemented at the application level.
    
### Purchasing Options

One of the big differences between managing Virtual Machine Instances in EC2,
and using an On-Premises Hypervisor platform, is that the Purchasing Options of
the EC2 Instances become a meaningful concern. As AWS is so massively scaled,
there are quite a few options that can be selected to meet the closest needs for
any given application. Ranging from purchasing dedicated bare-metal servers to
fully use for dedicated VMs on underlying dedicated hardware, to bidding on
ephemeral compute with no expected availability.

- On-Demand Instances

  On-Demand is the most common way that EC2 Instances are created and billed.
  At any given time, you can create an Instance to use, and `Stop`/`Start`,
  `Restart`, or `Terminate` whenever it is no longer needed. You are only billed
  for time the Instance is in the `Running` or `Restart` state, so systems can
  be provisioned and destroyed freely as needed. Billing depends on the specific
  Operating System in use, and either accrues on a per second, minute, or hour
  basis, with each `Start` state transition accruing at least a minute.
  
  There is a default limit on the maximum of On-Demand Instances that can exist
  in an AWS account, but these values can be raised to a certain extent.
  
  Other that these potential problematic caveats, On-Demand billing should be
  the Purchasing Option of choice to start off with on any new project. Once the
  project scope has been worked out to the point where further performance and
  cost optimizations can be made, other billing modes can be more easily
  considered.
  
- Reserved Instances

  Reserved Instances are good to use if there are clear and well forecasted
  compute needs for an extended period of time. Reserved Instances allow for all
  the same usages as On-Demand, but do require up front commitments or either
  1-year or 3-year purchases. Once you have purchased a Reserved Instance
  allocation, you are billed for the chosen time whether the Instance is
  `Running` or not. The upside is that there is significant cost savings(quoted
  as up to 72% of On-Demand), so it is a worthwhile decision for well established
  projects.
  
  Reserved Instances do have some subtypes that give more flexibility on chosen
  Instance Types and ability to resize, but do come with associated cost
  increases.
  
  These Instances are provisioned the same way as you would provision an
  On-Demand Instance. When you provision the On-Demand Instance, any remaining
  Quota you have for Reserved Instances will be selected. If you provision more
  than the Reserved Quota you have available, or provision for Types that don't
  have Quota in reserve, the Instances will be created at On-Demand prices. And
  if you have additional Quota left unused, you are still paying for that
  capacity.
  
- Spot Instance

  One beneficial item that comes with the massive scale of AWS, is that there is
  frequently under-utilization of resources during non-peak times. AWS attempts
  to approach maximum utilization here by providing what are known as Spot
  Instances. When you chose to provision a new EC2 Instance, you have to option
  to select a Spot Instance, and specify the maximum rate you are willing to
  pay. Depending on resource supply and demand, and current Spot Price bids,
  your instance will either be created or fail. If you have a Spot Instance
  `Running`, and the current Spot Price raises above your set maximum, your
  instance will `Terminate` with a two minute warning.
  
  There is no guarantee of Spot Instance availability unless you are always
  willing to pay current Spot Prices. Also note, Spot Instances have less
  priority that On-Demand Instances, which have less than Reserved instances. If
  AWS as a whole have a resource crunch, resource options become unavailable
  accordingly.
  
  You can probably see how this Purchasing type could lead to massive cost
  savings for some appropriate applications, but it does necessitate some more
  design considerations that could complicate the design greatly.
  
- Dedicated Instances/Hosts

  The last option to allocate EC2 resources is with Dedicated Instances and
  Hosts. These two options do have some difference in the specifics of use, but
  they are more similar than different. With all the previous options, when you
  provision an Instance it gets placed into Shared Hypervisor Clusters that
  share the underlying bare-metal resources with all other AWS users. While this
  setup is secure and performant in most cases, there are situations in which
  dedicated bare-metal resources are required per Account or Tenant. This could
  be due to security or regulatory concerns(e.g. banking), where non-zero risk
  of Hypervisor exploits in multi-tenant environments may not be acceptable; or
  high performance/low latency applications where intricacies with unknown
  multi-tenant scheduling might cause unacceptable performance drops.
  
  With Dedicated Instances, all Instances run on underlying un-shared Hypervisor
  infrastructure, where Instances from one AWS account will never share
  bare-metal hardware with Instances from another account. Hypervisor Scheduling
  to ensure this is less efficient, so it has a likewise increased cost, with an
  additional flat fee per hour per region, alongside the On-Demand or Reserved
  or Spot fees.
  
  This control is taken one step further with Dedicated Hosts, where you get
  sole access to individual Hypervisor servers. You have the capability to
  manually schedule VM Instances onto these Hosts, with a little more control
  over the specifics of the scheduling. This can come in handy for cases such as
  licensing of node-locked software, or manual scheduling of
  high-performance/high-security applications where cross contention of
  resources is unacceptable.


## Auto-Scaling

A large benefit of the AWS EC2 platform is that it allows for easy resource
scaling of services. Normally with bare-metal servers or low-feature
Hypervisors, it would be a manually process to either scale vertically by
replacing physical or virtual servers, or horizontally by manually adding and
configuring new servers as needed. EC2 offers this as a native feature, by
supporting horizontal auto-scaling of applications that relying on backend
Instances.

- Auto-Scaling Groups

  Each application service can be configured for Auto-Scaling by utilizing an
  Auto-Scaling group. Many AWS services can automatically make use of an
  Auto-Scaling group as their backend, and automatically split requests between
  any member Instances in the group as it scales in count up or down.
  
  We will be looking at the case of an AWS Network Load Balancer with Auto-Scaling
  groups in an upcoming section.
  
- Auto-Scaling Templates

  While Auto-Scaling Groups are the components that other resources interact
  with for routing to backends, the Auto-Scaling Templates are the components
  used in specifying how individual Instances are created and started. Where
  there is the capability to manually create individual EC2 Instances using a
  GUI or CLI, and configuring as needed before adding into service, Templates
  provide all the functionality needed to launch and manage an Instance
  automatically for production readiness.
  
- Auto-Scaling Options

  Along with services being fully managed and routed automatically, the
  Auto-Scaling Options give the ability to specify the logic around how and when
  resources should be scaled. Types provided are:
  
  - Manual
  
    Manually triggering scaling of resources

  - Scheduled
  
    Scaling based on time or date
    
  - Demand
  
    Scaling based on resource utilization
    
  - Predictive
  
    A combination of `Scheduled` and `Demand` to predict when Scaling is needed


## Networking

Along with the compute side of Virtual Machines, there is also a networking
component provided by EC2. Individual Instances need to be booted onto a virtual
network interface, with virtual switching so that Instances can communicate with
one another, and built-in DNS and DHCP so that routing, intranet, and internet
access can be setup automatically. Many of these systems come setup with sane
defaults in the default Virtual Private Cloud(VPC), and can be customized to
meet the needs of any project.

The most critical and frequently touched network configurations for EC2
instances are Security Groups and Elastic/Public IPs.

- Security Groups

  EC2 Security Groups implement Firewall rules for your individual Instances.
  You can specify individual Ingress and Egress rules to implement access for
  common needed resources and network access.
  
  By default, Instances get created with minimal network access, so this access
  will need to be opened for specific application needs.
  
- Elastic/Public IPs

  In the default VPC created, Instances get a Private IP Address assigned at the
  OS level. This will be an IP Address from a private 
  range([RFC 1918](https://www.rfc-editor.org/rfc/rfc1918#section-3)), that
  cannot be routed to the Internet. However, the default VPC will also assign
  Internet addressable Public IPs to each Instance, and setup network routing
  outside of the Instance OS so that Internet traffic can transparently navigate
  into and out of the Instance. This Public IP is an ephemeral allocation, so
  once an Instance is `Stopped`/`Started`, the IP will change.
  
  In many cases this will be sufficient, as other AWS resources(e.g. DNS) will
  be natively aware of this changing IP. But, when dealing with other external
  systems you are also able to request a static unchanging Elastic IP that can
  be bound to individual instances. These Elastic IPs will be bound to an
  account, and can be moved between Instances at will.
    

## AWS Regions and Availability Zones

All these EC2 offerings are available globally. As of this writing, AWS is
available in 30 regions spanning the globe, with multiple redundant Data Centers
in each for 96 global Availability Zones. There are also smaller locations
available for more specific Edge and Regional use cases, but we will not be
covering those to any great extent in this course.

- Regions

  AWS Regions are completely separate environments. The AWS Web Console and CLI
  separates the account selection of resources by Region, so Regions are
  intended to be used as completely independent environments.
  
  Major use cases for AWS Region selection would be compliance reasons, like
  hosting data in Regions with relevant laws, and performance, like co-locating
  Regional mirrors of applications globally for their intended geographical
  customer userbase.
  
- Availability Zones

  Whereas AWS Regions are implemented to be separate Cloud environments,
  Availability Zones(AZ) are intended to be used for High-Availability.
  Availability Zones are implemented as separate Data Centers in given Regions,
  so than common large disasters(think Electrical Substation outages) have the
  least chance of taking an entire Region offline. As they are intended
  for High-Availability, they are included as part of the Region resource
  display, and when designing applications you can specify to have some
  resources in one AZ, while having others in another.
  
  Common designs would be to have Active-Passive application designs split
  across an AZ, with fail-over triggered if one AZ were to fail.
   
   
## Instance Types

When provisioning a new Instance, one of the core requirements needed is an
understanding of the resource requirements for a given system. A good rule of
thumb is that keeping compute resources under 80% utilization is a good number
to aim for. As resource utilization raises beyond that, most systems start to
degrade in performance.

With EC2, unlike common consumer Hypervisors like VirtualBox and VMware, AWS
doesn't allow for complete customization of VM hardware allocated. Instead,
there are a common set of Instance Type sizings which can be selected for a
given use case. While this doesn't allow for perfect customization, EC2 Instance
Types are available in a large range of resource availabilities that will meet
most needs. Of course with prices to match the specifications.

- vCPU

  Virtual CPU Cores attached to the Virtual Machine Instance.

- Memory

  Allocated Random Access Memory(RAM) for the Virtual Machine Instance.
  
- Architecture

  Given CPU Architecture type for the Virtual Machine Instance. This will mainly
  be x86_64 for most usage, but 64-bit ARM Architectures are available as well.

- Storage

  Storage is its own topic; for here we are considering Root Disk storage.
  Specifies the size and speed of Elastic Block Store(EBS) backed, or Instance
  backed Root Disk storage.

- Network Bandwidth

  Specifies the Network Bandwidth allocated for the given Type.

- Other

  Some Types are purpose designed for specific workloads; like attached GPU
  cards, or other types of accelerator cards.

Depending on the specifics of a given application, Instance Type can be chosen
to meet the best Pricing to Performance. For most if not all the lab work, we
will be making use of the `t2.micro` Type, which has
1 vCPU/1GiB Memory/8GiB Root Disk, and is free up to a certain usage for the
Free Tier Account.


## Amazon Machine Image

With other common Virtual Machine software like VirtualBox or VMware Fusion, it
is common to provision the Virtual Machine Instance virtual hardware, and then
install an Operating System from an ISO Disk Image installer. When creating
Instances in EC2, this ISO install process is explicitly disallowed. Instead,
the base Operating System is managed as a hard drive Disk Image.

In the case of EC2, the Amazon Machine Image(AMI) disk image format is used.
There are a wide variety of premade OS Disk Images provided to choose from, and
you can also upload you own, or create a point-in-time snapshot from a current
Instance to use as a backing Machine Image. When we are starting from pre-built
base images with EC2, provisioning of Instances is quite fast, and applications
can even be launched straight from application specific pre-built Disk Images.
All dependencies, software, and launch processes can be embedded directly into
the image. The process of generating these custom Disk Images is generally its
own complex automated workflow.


## Instance Access and Bootstrapping

When provisioning an Instance, there are some initial required bootstrapping
steps that need to be taken, which should not be embedded directly into a Disk
Image. There are many ways to solve this problem, but some common ones are
natively provided by EC2.

- Access

  When you first install a Linux Operating System, an initial step is to setup a
  user and password, and optionally an ssh key pair for password-less access.
  These could be embedded directly into a base image, but that immediately runs
  into the issue of bad password management best practices.
  
  The system that EC2 implements to work around this is the automatic injection
  of ssh public key pairs, so that you can create or select a key pair for use
  at Instance creation. This allows for safer credential management, as well as
  certificate rotation possibilities.
  
  When trying to access an Instance once it is in the `Running` state, any ssh
  client will work as long as the necessary ssh private key component is
  provided during login.
  
- First Run Bootstrap

  In the case of many complex applications, there are processes that need to run
  before the intended application actually launches. A good example could be a
  Disk Image that is configured with the OS and all dependencies, but needs to
  sync data or the most up-to-date application code before launching. For these
  cases, EC2 supports "User Data" scripts that can be passed to the "cloud-init"
  application that is installed on almost all Cloud Images these days. Once the
  provisioning Instance enters the `Running` state for the first time, it will
  run scripts specified in the User Data field as the very first process. 


## Tags and Metadata

Tags and Metadata are fairly self explanatory, and can be completely arbitrary
in nature. They allow for attaching any arbitrary string values to Tag and
Metadata fields associated with any of the EC2 objects. This is mainly useful
for any sort or record keeping or comments, but could be used however they need
to be for custom Cloud Automation logic.
    

## Next Steps

This should be the very basics of EC2 needed to get started with creating
functioning VM environments. However, there is quite a bit more functionality
available to be used in the design of various complex Cloud environments.
The complexity does increases dramatically however as less standard
configurations are used, and exceptions are run up against.

The EC2 Documentation is the best place to start when trying to learn more, and
when tying to track down specific features and configuration options:

- [EC2 Documentation](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/concepts.html).

While most of this content explicitly covers Linux OS VM hosts, it does also
apply to a large extent for Windows resources, but with some additional caveats
and modifications needed.
