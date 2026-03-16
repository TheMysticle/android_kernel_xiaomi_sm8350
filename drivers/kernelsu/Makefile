ksu_obj-y := ksuinit.o
ksu_obj-y += allowlist.o
ksu_obj-y += app_profile.o
ksu_obj-y += apk_sign.o
ksu_obj-y += sucompat.o
ksu_obj-y += throne_tracker.o
ksu_obj-y += setuid_hook.o
ksu_obj-y += lsm_hook.o
ksu_obj-y += kernel_compat.o
ksu_obj-y += kernel_umount.o
ksu_obj-y += supercalls.o
ksu_obj-y += feature.o
ksu_obj-y += ksud.o
ksu_obj-y += file_wrapper.o
ksu_obj-y += selinux/selinux.o
ksu_obj-y += selinux/sepolicy.o
ksu_obj-y += selinux/rules.o

kernelsu-objs := $(ksu_obj-y)

ccflags-y += -I$(srctree)/security/selinux -I$(srctree)/security/selinux/include
ccflags-y += -I$(objtree)/security/selinux -include $(srctree)/include/uapi/asm-generic/errno.h

obj-$(CONFIG_KSU) += kernelsu.o

KSU_GIT_VERSION := 2967
KSU_GIT_TAG := v3.0.1
KSU_GIT_BRANCH := "susfs-rksu-master"
KSU_GIT_VERSION_VALID := 1

# ksu_version: major * 30000 + git version for historical reasons
ifdef KSU_GIT_VERSION_VALID
$(eval KSU_VERSION=$(shell expr 30000 + $(KSU_GIT_VERSION)))
$(info -- KernelSU version: $(KSU_VERSION))
ccflags-y += -DKSU_VERSION=$(KSU_VERSION)
else
$(warning "KSU_GIT_VERSION not defined! It is better to make KernelSU-Next a git repository!")
KSU_VERSION_FALLBACK := 1
$(info -- KernelSU version fallback: $(KSU_VERSION_FALLBACK))
ccflags-y += -DKSU_VERSION=$(KSU_VERSION_FALLBACK)
endif

ccflags-y += -DKSU_BRANCH=\"$(KSU_GIT_BRANCH)\"

ifdef KSU_GIT_VERSION_VALID
$(eval KSU_VERSION_TAG=$(KSU_GIT_TAG))
$(info -- KernelSU tag: $(KSU_VERSION_TAG))
ccflags-y += -DKSU_VERSION_TAG=\"$(KSU_VERSION_TAG)\"
else
$(warning "KSU_VERSION_TAG not defined! It is better to make KernelSU-Next a git submodule!")
KSU_VERSION_TAG_FALLBACK := v0.0.1
$(info -- KernelSU tag fallback: $(KSU_VERSION_TAG_FALLBACK))
ccflags-y += -DKSU_VERSION_TAG=\"$(KSU_VERSION_TAG_FALLBACK)\"
endif

# SELinux drivers check
ifeq ($(shell grep -q "current_sid(void)" $(srctree)/security/selinux/include/objsec.h; echo $$?),0)
ccflags-y += -DKSU_COMPAT_HAS_CURRENT_SID
endif

ifeq ($(shell grep -q "struct selinux_state " $(srctree)/security/selinux/include/security.h; echo $$?),0)
ccflags-y += -DKSU_COMPAT_HAS_SELINUX_STATE
endif

# Handle optional backports
ifeq ($(shell grep -q "strncpy_from_user_nofault" $(srctree)/include/linux/uaccess.h; echo $$?),0)
ccflags-y += -DKSU_OPTIONAL_STRNCPY
endif

ifeq ($(shell grep -q "ssize_t kernel_read" $(srctree)/fs/read_write.c; echo $$?),0)
ccflags-y += -DKSU_OPTIONAL_KERNEL_READ
endif

ifeq ($(shell grep "ssize_t kernel_write" $(srctree)/fs/read_write.c | grep -q "const void" ; echo $$?),0)
ccflags-y += -DKSU_OPTIONAL_KERNEL_WRITE
endif

ifeq ($(shell grep -q "int\s\+path_umount" $(srctree)/fs/namespace.c; echo $$?),0)
ccflags-y += -DKSU_HAS_PATH_UMOUNT
endif

ifeq ($(shell grep -q "inode_security_struct\s\+\*selinux_inode" $(srctree)/security/selinux/include/objsec.h; echo $$?),0)
ccflags-y += -DKSU_OPTIONAL_SELINUX_INODE
endif

ifeq ($(shell grep -q "task_security_struct\s\+\*selinux_cred" $(srctree)/security/selinux/include/objsec.h; echo $$?),0)
ccflags-y += -DKSU_OPTIONAL_SELINUX_CRED
endif

ifeq ($(shell grep -q "atomic_t\s\+filter_count" $(srctree)/include/linux/seccomp.h; echo $$?),0)
ccflags-y += -DKSU_OPTIONAL_SECCOMP_FILTER_CNT
endif

# some old kernel backport this, let's check if put_seccomp_filter still exist
ifneq ($(shell grep -wq "put_seccomp_filter" $(srctree)/kernel/seccomp.c $(srctree)/include/linux/seccomp.h; echo $$?),0)
ifeq ($(shell grep -wq "seccomp_filter_release" $(srctree)/kernel/seccomp.c $(srctree)/include/linux/seccomp.h; echo $$?),0)
ccflags-y += -DKSU_OPTIONAL_SECCOMP_FILTER_RELEASE
endif
endif

ifeq ($(shell grep -q "anon_inode_getfd_secure" $(srctree)/fs/anon_inodes.c; echo $$?),0)
ccflags-y += -DKSU_HAS_GETFD_SECURE
endif

ifeq ($(shell grep -q "security_inode_init_security_anon" $(srctree)/include/linux/security.h; echo $$?),0)
ccflags-y += -DKSU_COMPAT_HAS_INIT_SEC_ANON
endif

# Checks Samsung
ifeq ($(shell grep -q "CONFIG_KDP_CRED" $(srctree)/kernel/cred.c; echo $$?),0)
ccflags-y += -DSAMSUNG_UH_DRIVER_EXIST
endif
ifeq ($(shell grep -q "SEC_SELINUX_PORTING_COMMON" $(srctree)/security/selinux/avc.c; echo $$?),0)
ccflags-y += -DSAMSUNG_SELINUX_PORTING
endif

# Custom Signs
ifdef KSU_EXPECTED_SIZE
ccflags-y += -DEXPECTED_SIZE=$(KSU_EXPECTED_SIZE)
$(info -- Custom KernelSU Manager signature size: $(KSU_EXPECTED_SIZE))
endif

ifdef KSU_EXPECTED_HASH
ccflags-y += -DEXPECTED_HASH=\"$(KSU_EXPECTED_HASH)\"
$(info -- Custom KernelSU Manager signature hash: $(KSU_EXPECTED_HASH))
endif

ifdef KSU_MANAGER_PACKAGE
ccflags-y += -DKSU_MANAGER_PACKAGE=\"$(KSU_MANAGER_PACKAGE)\"
$(info -- KernelSU Manager package name: $(KSU_MANAGER_PACKAGE))
endif

$(info -- Supported KernelSU Manager(s): tiann, rsuntk, 5ec1cff, KOWX712, WildKSU, KSUNEXT)

ccflags-y += -Wno-strict-prototypes -Wno-int-conversion -Wno-gcc-compat
ccflags-y += -Wno-declaration-after-statement -Wno-unused-function -Wno-missing-prototypes

## For susfs stuff ##
ifeq ($(shell test -e $(srctree)/fs/susfs.c; echo $$?),0)
$(eval SUSFS_VERSION=$(shell cat $(srctree)/include/linux/susfs.h | grep -E '^#define SUSFS_VERSION' | cut -d' ' -f3 | sed 's/"//g'))
$(info -- KernelSU-SusFS version: $(SUSFS_VERSION))
else
$(error -- You have not integrated susfs in your kernel yet. Read: https://gitlab.com/simonpunk/susfs4ksu)
endif

# Keep a new line here !! Because someone may append config