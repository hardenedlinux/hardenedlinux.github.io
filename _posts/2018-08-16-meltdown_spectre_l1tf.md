---
layout:         post
title:          Nightmares never goes away
summary:        Meltdown, Spectre variants, L1TF. This is the year of side channel party for x86.
categories:     system-security
---


by Shawn C[ a.k.a "citypw"]

## Meltdown/Spectre

Google project zero's [write-up](https://googleprojectzero.blogspot.com/2018/01/reading-privileged-memory-with-side.html) explains how the vulnerablities( meltdown, spectre v1/v2) work. More info about v3a and v4, check Google project zero's [bug tracker](https://bugs.chromium.org/p/project-zero/issues/detail?id=1528) and [INTEL-SA-00115](https://www.intel.com/content/www/us/en/security-center/advisory/intel-sa-00115.html).

| Vulnerablity                                        | Affect                 | Kernel mitigation   | Compiler support |
|:---------------------------------------------------:|:----------------------:|:-------------------:|:----------------:|
| v3 Meltdown( rogue data cache load (CVE-2017-5754))    | < IceLake( 2018/2019) | KPTI/[PAX_UDEREF](https://github.com/hardenedlinux/grsecurity-101-tutorials/blob/master/grsec-code-analysis/PAX_MEMORY_UDEREF.md)     | N/A              |
| Spectre v1( bounds check bypass (CVE-2017-5753))    | < IceLake( 2018/2019) | [Code hardening](https://lwn.net/Articles/746551/) | N/A              |
| [Spectre v1.1](https://people.csail.mit.edu/vlk/spectre11.pdf) ( Bounds check bypass on stores( CVE-2018-3693))    | < IceLake( 2018/2019) | ? | N/A              |
| [Spectre v1.2](https://people.csail.mit.edu/vlk/spectre11.pdf) ( Read-only protection bypass)    | < IceLake( 2018/2019) | ? | N/A              |
| Spectre v2( branch target injection (CVE-2017-5715))| < IceLake( 2018/2019) | RETPOLINE/[IBPB+IBRS](https://newsroom.intel.com/wp-content/uploads/sites/11/2018/01/Intel-Analysis-of-Speculative-Execution-Side-Channels.pdf) | >= [GCC 7.3](https://gcc.gnu.org/ml/gcc/2018-01/msg00197.html), older version backport needed      |
| [v3a](https://www.us-cert.gov/ncas/alerts/TA18-141A)( CVE-2018-3640 – Rogue System Register Read (RSRE) | < IceLake( 2018/2019 | ? | ? |
| [Spectre v4](https://blogs.technet.microsoft.com/srd/2018/05/21/analysis-and-mitigation-of-speculative-store-bypass-cve-2018-3639/) CVE-2018-3639 – Speculative Store Bypass (SSB) | < IceLake( 2018/2019 | microcode for now | ? |
| [Lazy FP state restore](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2018-3665) CVE-2018-3665 | < IceLake( 2018/2019 | microcode for now | ? |
| [L1TF](https://software.intel.com/security-software-guidance/software-guidance/l1-terminal-fault) ( [CVE-2018-3615, CVE-2018-3620,CVE-2018-3646/INTEL-SA-00161](https://software.intel.com/security-software-guidance/insights/deep-dive-intel-analysis-l1-terminal-fault)) | < IceLake( 2018/2019) | ? | N/A |

Check the CPU status at runtime:
* /sys/devices/system/cpu/vulnerabilities/meltdown
* /sys/devices/system/cpu/vulnerabilities/spectre_v1
* /sys/devices/system/cpu/vulnerabilities/spectre_v2
* /sys/devices/system/cpu/vulnerabilities/spec_store_bypass
* /sys/devices/system/cpu/vulnerabilities/l1tf


## Kernel maintainence assessment( Meltdown/Spectre v1/v2)

| Kernel             | Difficulty level to fix  | Description         | Support version? |
|:------------------:|:-----------------:|:-------------------:|:----------------:|
| PaX/Grsecurity     | Nightmare    | Multiple features( out-of-tree mostly) must be adapted | 3.14 -- 4.14     |
| GCP/AWS/Azures     | Hardcore           | Multiple features? | 2.6.x -- ?       |
| GNU/Linux distro   | Hurt Me Plenty    | Closer to the upstream except RHEL/Centos    | 3.10 -- 4.14     |
| Alibaba            | Bring It On       | A few non-security features must be adapted   | 2.6.x -- 4.4     |
| Tencent            | I Can Win          | Pretty much the same as CentOS | 2.6.x -- 3.x?    |

* PaX/Grsecurity, the current features( USERCOPY/KERNEXEC/UDEREF/RAP/RANDSTRUCT/etc) need a lot of modifications to fit with the changes for the fixes from spectre v1/v2( RETPOLINE?) and especially the KPTI infrastruture. Only a few maintainers can do the job, which is so obviously. 

* [GCP](https://cloud.google.com/security/cpu-vulnerabilities/), GCP has done the good job about the whole process( risk assessment, research/dev about RETPOLINE and the deployment) for their customers. It's a heavy workload for GCP/AWS/Azures maintainers due to ( highly customized?) linux kernel and the changes from KPTI infrastructure.

* The major GNU/Linux distro fixed Meltdown by backporting the KPTI. RHEL is [temporarily stopped provide the packages](https://access.redhat.com/solutions/3315431) of CPU microcode update( IBRS?) which supposed to address [Spectre](https://access.redhat.com/security/vulnerabilities/speculativeexecution) v2 but it causes the boot issues.

* The maintainers from Alibaba/Tencent doesn't worry too much about the backport work. Alibaba had customized kernel in some old kernel trees( 2.6.x) but their current maintainence kernel is v4.4( Dec 2017) based due to policy changed to make their production kernel closer to the mainline. Tencent( TLinux)'s linux kernel is pretty much the same as the distro kernel( CentOS).


## Situational hardening vs. OpsDEV

| Defense solution                             | Security      | Deployment level    | Performance hit  | Description           |
|:--------------------------------------------:|:-------------:|:-------------------:|:----------------:|:---------------------:|
| TSX/page fault detector                      | Low           | Easy                | None         | Regular attack surfaces on Meltdown|
| KPTI/RETPOLINE                               | Medium        | Hard                | High?             | Meltdown, Spectre v2 |
| PaX/Grsecurity( 2018 version)                | High          | Easy                | High?            | Meltdown, Spectre v1/v2 |

The hardware's debts paid by the software now. Meltdown/Spectre are CPU design flaws that hard to be fixed for the current hardware( until Icelake?). The current proposal solution will cause the performance hits, e.g: KPTI could be the serious performance impact in heavily I/O operations. Some customers doesn't even plan to fix Meltdown via KPTI because the performance hits, while others doesn't have the proper technical/risk assessment. IMOHO, Meltdown/Spectre doesn't have the general solution. I'm not going to discuss those Spectre variants which only can be resolved by microcode. Let's try to figure out the possible solution for the different requirement.

* Low-cost( low security & easy to deployment): According to the public exploits, Meltdown seems more danger than Spectre. So this proposal hardening solution won't contain the mitigation for Spectre. If security critiria is only prevent the massive exploitation w/o any performance hits, you may not need KAISER/KPTI. Be careful if you are intend to deploy page fault detector only. Seriously? Well, you can't solve Meltdown and Spectre variants at low cost.

* Standard solution( medium security & hard to deployment): KPTI( Linux mainline's solution merged in v4.15 and it's backport to v4.14) should be used for mitigate the Meltdown. [RETPOLINE](https://security.googleblog.com/2018/01/more-details-about-mitigations-for-cpu_4.html) is a performance-oriented solution and it benefits the pre-Skylake machines. But it increases the [burden of software deployment](https://cyber.wtf/2018/02/13/in-debt-to-retpoline/) since the retpoline needs the re-compile the software for the new CPU. RETPOLINE is added to be support since GCC 7.3 and it may be backport to GCC 6.x. RETPOLINE is the best option since IBRS/IBPB microcode is not ready for now( Feb 16 2018). Most GNU/Linux distro( Debian/SuSE/RedHat/etc) and cloud vendors( including GCP/AWS/Azures) will choose this path.

* Best trade-off solution( high security & easy to deployment): PaX's UDEREF is the best one for both security and performance. This feature was designed by PaX team to kill [ret2usr attack](https://github.com/hardenedlinux/grsecurity-101-tutorials/blob/master/kernel_mitigation.md#ret2usr-protection) in the 1st place but it brings more layers of defense even better than hardware implementation( e.g: SMAP on >= Broadwell). SMAP-enhanced PaX's UDEREF made the performance improvement since [PaX/Grsecurity v4.12](https://grsecurity.net/grsecurity_4_12_updates.php). PaX's UDEREF implemented the per-cpu pgd back in 2013, which make it easy to implement the full kernel page isolation. The SMAP-enhanced UDREF fixed Meltdown in v4.14.13 and the performance is better than KPTI, while it also provide more security features...


## L1TF

L1TF is a new beast can perform side channel attack in Intel CPU by triggering unmapped memory resulted in a terminal fault where is L1TF comes into play. L1TF affects multiple levels of system software, including OS, SMM, Hypervisor/VMM and SGX:

| Vulnerablity                       | Impact        |  Description      |
|:----------------------------------:|:-------------:|:----------------------:|
| CVE-2018-3646                      | VMM           | malicious guest OS is able to gain the data( shared L1D) from other guest OS or VMM itself |
| CVE-2018-3620                      | OS/SMM        | malicious user is able to gain the priviledge user's data or priviledged user is able to gain the data from SMRAM |
| CVE-2018-3615                      | SGX           | Enclave is able to launch the attack to the other enclaves |

The current mitigation is a new MSR( IA32_FLUSH_CMD, 0x10B) introduced by Intel. OS/SMM/VMM/SGX can utilize it to flush L1 cache only at runtime which avoid the performance hit. Btw, cloud vendors can isolate the vCORE for each VM by utilizing the CPU affinity. It seems only [GCP has done it](https://cloud.google.com/blog/products/gcp/protecting-against-the-new-l1tf-speculative-vulnerabilities) before the full disclosure according to the public info. 


## Defensive side in cy..b.errr..

The core infrastructures( SMM( part of firmware), SGX( supported by microcode, BIOS/UEFI firmware and Intel ME), OS and hypervisor) paid the price for CPU bugs. It's also a challenge for the operations at data center. How many Spectre variants do we have for now? 7? Ok, the magic number will continue to grow as always. It's been tough for those security engineering/consultant work w/ data center to deal w/ side channel vulnerablities in past years. Risk assessment, vulnerablity anaylsis, patch review, regression testing for important applications, upgrade failures and other bunch of work to deal w/ both ppl and technical issues. Assuming that the security experts can [tune the system](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/plain/Documentation/admin-guide/l1tf.rst)( e.g: L1TF) well but we're likely to go through the whole process again( and again in the future?). What if firmware upgrade failures in an old enough machine doesn't compliant w/ [NIST SP-193](https://csrc.nist.gov/publications/detail/sp/800-193/final)? Oh, you can re-flash it by SPI programmer manually but how about hundre thousands of machines error occurs. The majority of security industry may not like to develop the mitigation/prevention( it's hard, I know!). Tons of money spent on a bunch of fancy "fastfood" shows in some vendorcons each year only make things worse. Do we still remember those [Intel ME vulnerablities](https://github.com/hardenedlinux/firmware-anatomy/blob/master/hack_ME/me_info.md) not long ago? Intel runs MINIX3-based OS on an old good independent x86 CPU since Skylake( 2015) and it seems doesn't even have the basic mitigation which exists already in 90s. How about the old bug from [TXT/TPM](https://www.ssi.gouv.fr/uploads/IMG/pdf/Cansec_final.pdf)( really neat one)? How many of us are willing to implement some promising "cloud" security features( Remote attestation?) based on TXT/TPMv2 w/ hardened the known attack surfaces by default? Well, not everything should be moved into the enclave. L1TF's vairant E2E doesn't make SGX look as Intel promised, see? I personally love enclave-based solution and I kind of like the SGX( except it's not open source and can't be audited properly) but why the hell everyone want to treat SGX like a silver bullet to sovle all security issues? Because of DLT ppl( did I say blockchain? fuck blockchain...) are so rush to solve their trustworthy issue? Well, we may still need to try harder to hardening the COREs. Let's make the COREs hardened!( again?)


## Tools

* [Spectre & Meltdown vulnerability/mitigation checker for Linux](https://github.com/speed47/spectre-meltdown-checker)
