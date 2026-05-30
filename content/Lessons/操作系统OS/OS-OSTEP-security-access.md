# Access Control
我们主要有两个目标
- 确定 request 是否符合我们的安全政策 (security policy)
	- referred to as **access control**
- 如果符合，执行操作。如果不符合，确保它没有被执行

example:

```bash
open("/var/foo", O_RDWR)
```

*At the high level, access control is usually spoken of in terms of* subjects, objects and access.

> [!Note] what is access control?
> An access control decision is *about whether a particular subject is allowed to perform a particular mode of access on a particular object*

- 谁(subject)能不能在另一个谁(object)上做某种 access
	- **particular** is hard to understand


如何很好的做到这一点？
- give subjects objects that belong only to them
**答案**： Virtualization
- Allows us to create virtual objects of this kind.
- Virtual memory 是一个很好的说明 access control 设计的例子

虚拟化是永远的答案，进程看起来 belong only 的 memory 实际是共享的，所以我们需要小心设计 
- 看起来独属于自己，不一定独属于自己

So merely relying on virtualization to ensure proper access just pushes the problem down to protecting the virtualization functionality of the OS

> [!Tip] Two basic approaches to solve this question
> - access control list
> - capabilities
## Access control list system

ACLs for access control

- *Each file* has its own access control list, resulting in simpler shorter lists and quicker access control checks
- 对事不对人

- Good points:
	- easy to figure out *who* is allowed to access a resource
	- 想要改可以对 object access 的 subjects 集合，仅需修改 `ACL`
	- ACL is typically kept either with or near the file itself，容易携带的 meta information
		- 特别对于 distributed system 友好
- Less desirable features
	- 成也败也：需要存储额外的 list 信息，有时候可能很长
	- Hard to figure out the *entire* set of resources some principal (a process or a user) is permitted to access. 
		- need to check every single ACL
	- 在 distributed environment，需要有 common view of identity across all the machine for ACLs to be effective
		- 因为可能访问其他机器的文件



> [!Note] Aside: Name Space
> 名字空间的设计本质上是在全局唯一性和管理成本之间做权衡。
> **分布式系统上**：怎么确定一个 `/etc/password` 是 same的文件还是什么？
> - One approach: do not to bother and to understand
> 	- process IDs
>  - Another approach: to require an authority to approve name selection. **AFS**
>  - Another approach: **hand out portions** of the name space to each participant and allow them to assign any name from that portion
> 	- WWW, IPv4


# Capabilities for Access Control

More like **keys** or **tickets**
How it does?
- When the call (*like `open`*) is made, either your **application** would provide a capability permitting your process to open the file in question as a parameter, or the **operating system** would find the capability for you.
- 无论哪种情况，OS 都会检查 capability

> [!Question] question about Capabilities
> - What, precisely, is a **capability**?
> - How does the OS check the validity of capability?
> - Where do capabilities come from, in the first place?

**capabilities** are bunches of bits. They are data
- **造成了一个问题**：anyone can create any bunch of bits they want.
- if a process has one copy of a particular set of bits, it’s trivial to create more copies of it
	- capability 可以伪造
	- capability 可以复制，甚至到另一个机器上
	- 不安全
**解决方案**：放到 OS 内核（被保护的 memory）管理 capability

如果想要依赖 capabilities 来做 access control, OS 将需要维护他自己的受保护的 capability list (for each process)
o

if we want to rely on capabilities for access control, the operating system will need to maintain its own protected capability list for each process. That’s simple enough, since the OS already has a per-process protected data structure, the PCB. Slap a pointer to the capability list (stored in kernel memory) into the process’ PCB and you’re all set. Now when the process attempts to open /tmp/foo for read/write, the call traps to the OS, the OS consults the capability list for that process to see if there is a relevant capability for the operation on the list and proceeds accordingly


**另一个解决方案**
- 加密的能力 (Capabilities can be cryptographically protected)


## Good and Bad
- easy to determine which system resources a given principal can access
- determining the entire **set of principals** who can access a resource becomes more *expensive*

- Offer a good way to create processes with limited privileges
	- Just write the *capabilities* of it
		- 甚至可以选择 X，Y，Z 中的某两个，很简单
	- 但是对于 ACLs，这个 access control 的subset 的传递很困难。
	
# Summary of ACLs and Capabilities

在真实 Linux 中，他们混合使用 ACLs 和 Capabilities
- 初始打开用 ACL 鉴权
- 然后生成一个 data structure, 挂载在 PCB 上，再用的时候，只简单的用
