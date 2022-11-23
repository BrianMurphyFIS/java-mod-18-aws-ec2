# AWS EC2


## Learning Goals

- Understanding VM Instances
- Understanding EC2 Instance Types
- Understanding EC2 Images(AMI)
- Understanding Availability Zones
- Understanding Autoscaling Groups
- Understanding SSH Key Access and Management


Callback of Cloud topics
    Virtual Datacenter
    Servers/EC2 -> VMs/Servers
    Virtual Machines/Hypervisors(vmWare vSphere, VirtualBox, Xen, KVM, Windows(?))/Virtual Networking Components
    IaaS
        If you recall from our previous overview of Cloud, Cloud platforms can be though of to esentially take the place as a Virtual Datacenter. One of the core components of a functioning Datacenter, physical or virtual, is the concept of general purpose computation
        servers, without which no custom developed applications could run. In the case of AWS, this role is covered mainly by EC2(?). EC2 owns the creation and management of Virtual Machines(VM), both the core component of provisioning the underlying machine hardware,
        as well as all the ancilory pieces to allow these machines to operated at scale. I.E., networking components, scaling features, and distribution.
        If you have ever worked with a featured Hypervisor platform such as vmWare vSphere, EC2 offers many of the same allowances. If you've used a less featurful offering, such as VirtualBox, VmWare Horizon(?), KVM, etc., you've already been introduced to the main concepts
        behind hypervisors and Virtual Machines, but without the more enterprise required components of networking, scaling, failover, and clustering. In fact, EC2 relies on many of these same industry standard Hypervisor platforms(links), with EC2 being a user friendly
        wrapper integrating well with the rest of AWS.
        With all this being the case, EC2 falls quite easily into the category of Infrastructure as a Service(IaaS), and should be used for that case. Any situation which calls for the construction of custom infrastructure(servers, operating systems, networking), should
        be a potential candidate for EC2, whereas any higher abstracted software component needs might be more suited for other AWS offerings focused on Platform as a Service(PaaS) or SaaS(Software as a Service).


VM Instances
    What are instances
        The basic component of EC2 is the VM Instance. You can think of this in terms of a normal Virtual Machine, and in many cases it is quite similar to Virtual Machine Instances you might have used in other Hupervisor platforms. On EC2, you are charged for the runtime
        of any EC2 Instances in your account, and there is very little limit to the maximimum or minimum scales you are able to achieve. While we are not explicitly covering billing of resources, you should always keep this in mind, and keep an eye on the billing dashboards
        and limits, and an accidental choice could lead to hefty expences fast.
    State/Lifecycle of VM instances
        Instances have operational states and transitions that somewhat mimic the lifecycle of traditional physical servers or workstations. As you are managing these systems of the course of an application's development and operational lifecycle, any needed Instance might
        eventually pass through all these states. Most of these should be self-explanatory for anyone with even basic computor experience:
        - Start(Running)
          When you initially create an Instance, it will be created in a running state. The virtual hardware is powered on, the BIOS boots, and the Operating System starts running in that environment. The instance will be listed as "Running" once this process is all complete.
        - Stop(Stopped?)
          When a running instance is stopped, it will transition to the "Stopped" state. Stopping the instance can either be done from the running OS or the EC2 console/cli. Note that a VM stopped from the OS might potentially hang during shutdown, stalling or failing whatever
          operation is taking place. This needs to be planned for depending on the operations.
        - Reboot(Rebooting?)
          A reboot is essentially a Stop followed by a Start. The only real differentiating factor with this implementation in EC2 is that instance billing is handled differently between the two cases. With a Reboot there is no interruption, while a Stop/Start causes an
          interruption. This might matter greatly depending on the specifics of the instance types and OS images.
        - Delete(?)(Terminated?)
          Once an Image is no longer needed, it can be Deleted. Once deleted, it will be removed from usage, and set as "Terminated". It will remain visible in this state for an hour afterwards, as EC2 systems schedule cleanup. It cannot be recovered after the termination has
          been triggered(?). Note, this termination state might cause issues for automation/cli tooling. E.g. duplicate VM names could exist if one is in the terminated state.
        - Hibernate(?)
          Hibernate in EC2 is a similar concept to the Hibernate state common with most Operating Systems. The memory of the VM will be written out to disk, and then the Instance will be stopped. Once the Instance is started again, the memory contents is read in from disk,
          and the OS starts again from the same state where it left off. This state might be useful for infrequently used systems that require long pre-warm periods to load or build state/cache in memory. With Hibernate in EC2, you are not necessarily charged for the
          running time of the stopped Instance, but you are billed for the additional Disk space needed to store the memory contents. 
        - Recovery(Migration)
          Instance Recovery is a state that might be seen when EC2 underlying infrastructure fails. Migration of a Virtual Machine Instance from a failed Hypervisor server to another working cluster node is fully managed on the EC2 system side, but could lead to unavoidable
          downtime during the recovery. There is no native High Availability for single individual Instances(?), so redundancy still needs to be implemented at the application level.
    Purchasing Options
        One of the big differentiating factors between managing Virtual Machine Instances in EC2 and another Hypervisor platform, is that the Purchasing Options of the EC2 instances become a meaningful concern. As AWS is so massivly scaled, there are quite a few options
        that can be chosen to meet the closest needs for any given application. Ranging from purchacing dedicated bare-metal servers to fully use for dedicated VMs on underlying dedicated hardware, to bidding on ephemeral compute with no expected availability.
        - On-Demand Instances
          On-Demand is the most common way that EC2 Instances are created and billed. At any given time, you can create an Instance to use, and Stop/Start, Restart, or Terminate whenever it is no longer needed. You are billed for time the Instance is in the Start or Restart
          state, otherwise the systems can provisioned freely as needed. Billing depends on the specific Operating System in use, and either occurs on a per second, minute, or hour basis. With each Start state transition, there is at least a minute billed for. There is a default
          limit on the maximum of On-Demand Instances that can exist in an AWS account, but these values can be raised to a certain extent.
          Other that these potential problematic caviats, On-Demand billing should be the Purchasing Option of choice to start off with on an new project. Once the scope has been worked out more to the point where further performance/cost optimizations can be made, other billing
          modes can be moved to.
        - Reserved Instances
          Reserved Instances types are good modes to use if there is clear and well forcasted compute needs for a long period of time. Reserved Instances allow for all the same usages as On-Demand, but require up from commitment or either 1-year or 3(?)5-year purchases.
          Once you have purchased a Reserved Instance allocation, you are billed for the chosen time whether the Instance is Running or not. The upside is that there is significant cost savings, so it is a worthwhile decision for well established projects.
          Reserved Instances do have some subtypes that give more flexibility on chosen instance types, ability to cancel early()), and resizing, but do come with associated billing changes.
          These Instances are provisioned the same way as you would provision an On-Demand Instance. When you provision the On-Demand Instance, any remain Quota you have for Reserved Instances will be selected. If you provision more than the Reserved Quota you have available,
          or provision for types that don't match Reserved in reserve, the Instances will be created as On-Demand. And if you have additional Reserved in reserve, you are still paying.
        - Spot Instance
          One benificial item with AWS builiding a Cloud platform to scale to meet peek demands, is that the is frequently undersubscription of resources during non-peak times. AWS attempts to recuperate costs of this maintenance by providing what is known as Spot Instances.
          When you chose to provision a new EC2 Instance, you have to option to select a Spot Instance, and specify the maximum rate you are willing to pay. Depending in resource supply and demand, and current Spot Price bids, your instance will either be created, or it wont.
          If you have a Spot Instance running, and the current Spot Price raises above your set maximum, your instance will be terminated with a < 2 minute warning. One item to emphasize is that Instance lifecycle is fully tied to the Spot Price. There is no guarantee of
          availability unless you are always willing to pay current Spot Prices. Also note, that Spot Instances have less priority that On-Demand Instances, which have less than Reserved instances. If AWS as a whole have a resource crunch, resource options become unavailable
          accordingly. You can probably see how this Purchasing type could lead to massive cost savings for some appropriate applications, but it does necessitate some more design considerations that might complicate the environment design greatly.
        - Dedicated Instances/Hosts
          The last option to allocate EC2 resources is with Dedicated Instances and Hosts. These two options do have some difference in the specifics of use, but they are more similar than different.
          With all the previous options, when you provision and Instance, it gets placed into Shared Hypervisor Clusters that share underlying bare-metal resources with all other AWS users. While this setup is secure and performant in most general cases,
          there are situations in which dedicated bare-metal resources are required per Account/Client/Principal(?)/Tenant. This could be due to security or regulatory concerns(e.g. banking), where non-zero risk of Hypervisor exploits may not be acceptable in multi-tenant
          environments; or high performance/low latency applications where intrincities with unknown multi-tenant scheduling might cause unacceptable performance drops.
          With Dedicated Instances, all Instances run on underlying un-shared Hypervisor infrastructure, where Instances from one AWS account will never share bare-metal hardware with Instances from another account. Hypervisor Scheduling to ensure this is less effecient,
          so it has a likewise increased cost, with an additional flat fee per hour per region, alongside the On-Demand or Reserved or Spot fees.
          With Dedicated Hosts, this control is taken one step further, and you get sole access to individual Hypervisor servers. You have the capability to manually schedule VM Instances onto these Hosts, with a little more control over the specifics of the scheduling.
          This can come in handy for cases such as licensing of node-locked software, or manual scheduling of high-performance/high-security applications where cross contention of resources is unacceptable.
        on-demand, reserved, spot, dedicated hosts/instances
    Autoscaling
        A large benefit of the AWS EC2 platform, is that it allows for easy resource scaling of services. Normally with bare-metal servers or low-feature hypervisors, it would be a manually process to either scale vertically by replacing physical or virtual servers, or horzontally
        by manually adding and configuring new servers as needed. EC2 offers this as a native feature, which supports horizontal auto-scaling of applications that relying on backend Instances.
        - Groups
          Each service can be configured for auto-scaling by utilizing an autoscaling group. Many AWS services can automatically make use of an autoscaling group as their client(?), and automatically split requests amoung the members of the group as it scales in count up
          or down. We will be looking at the case of an AWS Network Load Balancer with autoscaling groups in an upcoming section.
        - Templates
          While Autoscaling Groups are the artifacts that other resources interact with for routing requests to backend resources, the Autoscaling Templates are the components used in specifying how individual instances are created and started. Where there is the capability
          to manually create individual EC2 instances using a GUI/CLI, and configuring as needed before adding into service, Templates provide all the functionality needed to launch an Instance, and manage the full process needed automaically to production ready.
        - Options
          Along with services being fully managed and routed automatically, the Autoscaling Options gives the ability to specify the logic around when and how resources should be scaled. Type provided are:
          - Manual: Manually triggering scaling
          - Scheduled: Scaling based on time or date
          - Demand: Scaling based on resource utilization
          - Predictive: A combination of Scheduled and Demand to predict when Scaling is needed
    Networking
        Along with the compute side of Virtual Machines, there is also a networking component. Individual Instances need to be booted onto a virtual network interface, with virtual switching so that Instances can communicate with one another, and built-in DNS and DHCP
        so that routing, intranet, and internet access can be setup automatically. Many of these systems come setup with sane defaults in the default Virtual Private Cloud(VPC), and can be customized to need.
        The most critical and frequently touched network configurations for EC2 instances are Security Groups and Elastic IPs.
        - Security Groups
          EC2 Security Groups implement Firewall rules for your individual Instances. You can specify individually ingress and egress rules to implement access for common needed resources and network access. By default, Instances get created with minimal access, so this
          access will need to be opened for specific application needs.
        - Elastic IPs/Public IPs
          In the default VPC created, Instances get a Private IP Address assigned at the OS level. This will be an IP Address from a private range(RFC ?), and cannot be routed to the internet. However, the default VPC will also assign Internet addressable Public IPs to
          each instance, and setup network routing outside of the Instance OS so that internet traffic can transparently navigate into and out of the Instance. This Public IP is an ephemeral allocation, so once an Instance is Stopped/Started, or many other cases, this IP
          will change. In many cases this will be sufficient, as other AWS resources will be natively aware of this changing IP, but when dealing with other external systems you are able to request a static unchanging Elastic IP that can be bound to individual instances.
          These Elastic IPs will be bound to an account, and can be moved between Instances at will.
    

    AWS Region / Availability Zones
        All these EC2 offerings are availably globally. As of this writing, AWS is available in 30 regions spanning the globe, with multiple redundant Data Centers in each for 96 global Availability Zones. There are also smaller locations available for more specific Edge
        and Regional use cases, but we will not be covering those to any great extent in this course.
        - Regions
          AWS regions are completely separate environments. The AWS console/GUI/CLI separates the accound selection of resources by region, and regions are intended to be used as independent environments. Major use cases for AWS Region selection would be compliance reasons,
          like hosting data in regions with relevant laws, and performance, like co-locating regional mirrors of application globally for customer userbase.
        - Availability Zones
          Whereas AWS regions are implemented to be separate Cloud environments, Availability Zones are intended to be used for High-Availability. Availability Zones are implemented as separate Data Centers in given Regions, so than common large disasters(think blown Electrical
          Substations) have the least chance of taking an entire Region offline. As they are intended for High-Availability, they are included as part of the Region resource display, and when designing applications you can specify to have some resources in one AZ, and some in
          another. Common designs would be to have Active-Passive application designs split across an AZ, with failover triggered if one AZ were to fail.
    
Instance Types
    When provisioning a new Instance, one of the core requirements needed is an understanding of the resource requirements. A good rule of thumb is that keeping compute resources under 80% is a good number to aim for. As resource utilization raises beyond that, most systems
    start to degrade in performance. Ultimately though, thourugh performance testing is needed to determine these explicit numbers.
    With EC2, unlike common consumer Hypervisors like VirtualBox, vmWare, AWS doesn't necessarily allow for complete customization of VM hardware allocated. Instead, there are a common set of Instance Type sizings which can selected for any given use case. While this isn't
    perfectly customizible, EC2 does have a large range of Types for various use cases, associated with prices to match.
    - vCPU: Virtual CPU Cores attached to the Virtual Machine Instance.
    - Memory: Allocated Random Access Memory(RAM) for the Virtual Machine Instance.
    - Architecture: Given CPU Architecture type for the Virtual Machine Instance. This will mainly be x86_64 for most usage, but 64-bit ARM Architectures are available as well
    - Storage: Storage is its own topic; for here we are considering Root Disk storage. Specifies the size and tier of EBS backed, or Instance backed Root Disk storage
    - Network Bandwidth: Specifies the Network Bandwidth allocated for the given Type
    - Other: Some Types are purpose designed for specific workloads, so may have attached GPU cards, or other types of accelerator cards.
    Depending on the specifics of a given application, Instance Type can be chosen to meet the best Pricing to Performance.
    For most if not all the lab work, we will be making use of the "t2.micro" Type, which has 1 vCPU/1GiB Memory, and is free up to a certain usage for the Free Tier Account.
    
Amazon Machine Image(AMI)
    With other common Virtual Machine software like VirtualBox or vmWare Fusion(?), it is common to provision the Virtual Machine Instance hardware, and then install an Operating System from an ISO disk image installer. When creating Instances in EC2, this is explicitly
    disallowed. Instead, the base Operating System is managed as a hard drive disk image. In the case of EC2, they use the AMI disk image. There are wide variety to premade OS images to choose from, and you can also upload you own, or create a point-in-time snapshot
    of a current Instance, and use that as a backing Machine Image.
    As we are starting from prebuilt base images here, provisioning of Instances is quite fast, and applications can even be launched straight from application specific pre-built Disk Images. All dependencies, software, and launch processes can be embedded directly into
    the image. The process of generating the initial disk image to use can be its own workflow.
    
Access/Key Pairs/Cloud Init
    When provisioning an Instance, there are some initial bootstraping components that need to be taken, which should not be embedded into a Disk Image. There are many ways to solve this problem, but some common ones are provided by EC2
    - Access
      When you first install a Linux Operating System, an initial step is to setup a user and password, and optionally an ssh key pair for passwordless access. These could be embedded directly into a base image, but that immediately runs into the issue of password management
      best practices. A system that EC2 implements, is the automatic injection of ssh public key pairs. At provisioning time, you can create a key pair, and select to inject any keys needed. This allows for safer credential management, and certificate rotation possibilities.
    - First Run Bootstrap
      In the case of many complex applications, there are processes that need to run before the intended application actually launches. A good example could be a Disk Image that is configured with all dependencies, but needs to sync data or most up-to-data application code
      before launching. For these cases, EC2 supports "User Data" scripts that can be passed to the "cloud-init" application that is installed on almost all Cloud Images these days. Once the provisioning Instance boots into the Template Disk Image, it will run scripts specified
      in the User Data field on first boot, running some initial steps before any other system processes run. 

Tags/Metadata
   Tags and Metadata are fairly self explanatory, and arbitrary. They allow for attaching any arbitrary string values to Tag and Metadata fields associated with any of the EC2 objects. This is mainly useful for any sort or record keeping or comments, but can be used
   however they need to be.
    
    
This should be the basics of EC2 needed to get started with creating functioning VM environments. However, the complexity increases dramatically as you work more towards solving specific problems. EC2 Documentation is the best place to start when trying to track down
very specific features and configuration options: (link)



