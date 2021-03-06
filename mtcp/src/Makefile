# TODO: Make this Makefile.in pretty

.PHONY: clean libccp

### TARGET ###
PS=0
DPDK=1
NETMAP=0
LRO=0
HWCSUM=1
MTCP_LIB_DIR=../lib
MTCP_LIB=libmtcp.a
MTCP_HDR_DIR=../include
MTCP_HDR = mtcp_api.h mtcp_epoll.h


### GCC ###
GCC=gcc-4.8

### FLAGS ###
ifeq ($(shell uname -m),x86_64)
GCC_OPT = -m64
else
GCC_OPT =
endif
GCC_OPT += -Wall -fPIC -fgnu89-inline -Werror
#DBG_OPT = -DDBGMSG -DDBGFUNC -DSTREAM -DSTATE -DTSTAT -DAPP -DEPOLL
#DBG_OPT = -DDBGMSG -DDBGFUNC -DSTREAM -DSTATE
#DBG_OPT += -DPKTDUMP
#DBG_OPT += -DDUMP_STREAM
#GCC_OPT += -g -DNETSTAT -DINFO -DDBGERR -DDBGCERR
GCC_OPT += -DNDEBUG -g -O3 -DNETSTAT -DINFO -DDBGERR -DDBGCERR -D__USRLIB__
#GCC_OPT += -DNDEBUG -g -DNETSTAT -DINFO -DDBGERR -DDBGCERR
GCC_OPT += $(DBG_OPT)

CFLAGS=-g -O2 -msse3 -msse4 -msse4.1 -msse4.2 -mavx2

ifeq ($(LRO),1)
GCC_OPT += -DENABLELRO
endif

ifeq ($(HWCSUM),0)
GCC_OPT += -DDISABLE_HWCSUM
endif

### LIBRARIES AND INCLUDES ###
INC_DIR=./include
INC= -I$(INC_DIR)

ifeq ($(PS),1)
LIBS = -lps
else
INC += -DDISABLE_PSIO
endif

ifeq ($(NETMAP),1)
# do nothing
else
INC += -DDISABLE_NETMAP
endif

# PacketShader LIBRARY and HEADER
PS_DIR=../../io_engine
INC += -I$(PS_DIR)/include

# DPDK LIBRARY and HEADER
DPDK_INC=/home/frank/mtcp/dpdk/include
DPDK_LIB=/home/frank/mtcp/dpdk/lib/
# CFLAGS for DPDK-related compilation
ifeq ($(DPDK), 1)
DPDK_MACHINE_FLAGS = $(shell cat /home/frank/mtcp/dpdk/include/cflags.txt)
INC += ${DPDK_MACHINE_FLAGS} -I${DPDK_INC} -include $(DPDK_INC)/rte_config.h
LIBCCP_PATH=/home/frank/mtcp/mtcp/src/libccp
LIBS = -L$(LIBCCP_PATH) -lccp
INC += -I$(LIBCCP_PATH)
else
INC += -DDISABLE_DPDK
endif

ifeq ($(wildcard /usr/lib/libhugetlbfs.so),) 
else
	GCC_OPT += -DHUGEPAGE
endif

### SOURCE CODE ###
SRCS = core.c tcp_stream.c config.c api.c eventpoll.c socket.c pipe.c \
	   tcp_util.c eth_in.c ip_in.c tcp_in.c eth_out.c ip_out.c tcp_out.c \
	   arp.c timer.c cpu.c rss.c addr_pool.c fhash.c memory_mgt.c logger.c debug.c \
	   tcp_rb_frag_queue.c tcp_ring_buffer.c tcp_send_buffer.c tcp_sb_queue.c tcp_stream_queue.c \
	   psio_module.c io_module.c dpdk_module.c netmap_module.c icmp.c \
		 ccp.c token_bucket.c

OBJS = $(patsubst %.c,%.o,$(SRCS))
DEPS = $(patsubst %.c,.%.d,$(SRCS))

ifeq ($V,) # no echo
	export MSG=@echo
	export HIDE=@
else
	export MSG=@\#
	export HIDE=
endif

### GOALS ###
all: default libccp

default: $(OBJS) $(MTCP_HDR)
	mkdir -p $(MTCP_LIB_DIR)
	ar rvs $(MTCP_LIB_DIR)/$(MTCP_LIB) $(OBJS)

libccp: 
	$(MAKE) -C ./libccp

$(OBJS): %.o: %.c Makefile
	$(MSG) "   CC $<"
	$(HIDE) $(GCC) $(CFLAGS) $(GCC_OPT) $(INC) $(LIBS) -c $< -o $@
$(DEPS): .%.d: %.c Makefile
	$(HIDE) $(GCC) $(GCC_OPT) $(INC) $(LIBS) -MM $(CFLAGS) $< > $@

-include $(DEPS)

$(MTCP_HDR):
	cp $(INC_DIR)/$@ $(MTCP_HDR_DIR)/$@

clean: clean-library
	rm -f *.o *~ core
	rm -f .*.d

clean-library:
	rm -f $(MTCP_LIB_DIR)/*
	rm -f $(MTCP_HDR_DIR)/*

distclean: clean
	rm -f Makefile
