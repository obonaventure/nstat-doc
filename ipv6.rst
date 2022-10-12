The IPv6 counters
=================

The IPv6 counters maintained by Linux can be divided four groups. The
first counters track the incoming and outgoing packets according
to the standard SNMP MIB. They are illustrated in the Case diagram
below.

.. code-block:: console
   :caption: IPv6 counters for incoming packets

                      Transport Layer
   -----------------------------------------------------
	                    /\
                            ||
			    ||
    Ip6InDelivers+++++++++++++++
                            ||
			    ||
			    |+-----------> Ip6InUnknownProtos
			    ||
			    ||
    Ip6InDiscards <---------+|
                            ||
			    |<----------------  Ip6ReasmOKs
			    ||                     /\     
			    ||   Ip6ReasmFails  <--+|
			    ||                     ||
			    |+--------------> Ip6ReasmReqds
			    ||
			    ||
    Ip6ForwDatagrams <------+|
                            ||
			    |+--------------> Ip6InAddrErrors
			    ||
			    ||
    Ip6InHdrErrors <--------+|
                            ||
			    ||
			    ||
			 +++++++++++++++++++ Ip6InReceives   
                            ||
   -----------------------------------------------------	  
                      Interface Layer
			 

When operating servers, it is interesting to check several of these
counters. First, the ``Ip6InAddrErrors`` and ``Ip6InHdrErrors`` should
remain at zero. The ``Ip6ForwDatagrams`` counter is only incremented on
routers. The ``Ip6ReasmReqds`` counts the number of IPv6 fragments that
the host receives. Most applications will avoid fragmentation and TCP
always uses Path MTU discovery. A TCP server should never receive
IPv6 fragments. If a TCP server receives such fragments, they
should be investigated, because it could be linked to some type of attack.
If a UDP server relies on packet fragmentation, you might want to reconsider
it and try to remove this requirement. Finally, ``Ip6InUnknownProtos`` should
also remain at 0 unless you support experimental protocols.
		      
.. Ip6InReceives                   4688796            0.0
.. Ip6InHdrErrors                  0                  0.0
.. Ip6InTooBigErrors               0                  0.0
.. Ip6InNoRoutes                   200                0.00
.. Ip6InAddrErrors                 0                  0.0
.. Ip6InUnknownProtos              0                  0.0
.. Ip6InTruncatedPkts              0                  0.0
.. Ip6InDiscards                   0                  0.0
.. Ip6InDelivers                   4638974            0.0
.. Ip6OutForwDatagrams             0                  0.0

Linux also supports the ``Ip6InTooBigErrors`` which is incremented
every time it attempts to transmit a packet larger than the interface's
MTU. This is usually a packet received over another interface that
needs to be forwarded, but could also be a locally generated packet 
   

In addition to these per packet counters, Linux tracks the number of
bytes received in the ``Ip6InOctets`` counter and the number
of bytes sent in the ``Ip6OutOctets`` counter.

.. Ip6InOctets                     28889881617        0.0
.. Ip6OutOctets                    28055522808        0.0

Three counters are maintained for the outgoing packets : ``Ip6OutRequests``,
``Ip6OutDiscards`` and ``Ip6NoRoutes``. Note that ``Ip6NoRoutes`` is only
incremented when a locally generated packet cannot be sent because there
is not route towards its destination. On a router, the ``Ip6InNoRoutes``
counter is incremented when a packet received on an interface cannot be
forwarded over another interface because there is no matching entry in
the routing table.

.. code-block:: console
   :caption: IPv6 counters for outgoing packets

                      Transport Layer
   -----------------------------------------------------
	                    
                            ||
			    ||
    Ip6OutRequest  +++++++++++++++
                            ||
			    ||
			    |+-----------> Ip6OutNoRoutes
			    ||
			    ||
    Ip6OutDiscards <--------+|
                            ||
                            \/
			    
   -----------------------------------------------------	  
                      Interface Layer   
   
.. Ip6OutRequests                  3534846            0.0
.. Ip6OutDiscards                  0                  0.0
.. Ip6OutNoRoutes                  13                 0.0


Fragmentation and reassembly
----------------------------

IPv6 hosts support fragmentation and reassembly, but in practice this
part of the protocol is and should rarely be used. TCP uses Path MTU discovery
and never relies on fragmentation. Other protocols such as UDP can use IPv6
fragmentation to send large packets, but applications should avoid relying
on IPv6 fragmentation. 

.. net.ipv4.ipfrag_high_thresh = 4194304                                       .. net.ipv4.ipfrag_low_thresh = 3145728                                        
.. net.ipv4.ipfrag_max_dist = 64                                               
.. net.ipv4.ipfrag_secret_interval = 0                                         
.. net.ipv4.ipfrag_time = 30   

.. net.ipv6.ip6frag_high_thresh = 4194304                                      
.. net.ipv6.ip6frag_low_thresh = 3145728                                       .. net.ipv6.ip6frag_secret_interval = 0                                        .. net.ipv6.ip6frag_time = 60   

.. todo: fragments or packets ?
   
The ``Ip6ReasmReqds`` counter tracks the number of fragments that were
received. The ``Ip6ReasmOKs`` counter is incremented when a packet has
been fully reassembled. Otherwise, the ``Ip6ReasmFails`` counter is
incremented. The ``Ip6ReasmTimeout`` tracks the expiration of the
IPv6 reassembly timer. By default, Linux keeps an IPv6 fragment during
up to 60 seconds in memory (this default can be changed with the
``net.ipv6.ip6frag_time`` ``sysctl`` variable). The memory used to
reassemble packets is restricted and can be configured using
the ``net.ipv6.ip6frag_low_thresh`` and ``net.ipv6.ip6frag_high_thresh``
``sysctl`` variables.

.. Ip6ReasmTimeout                 0                  0.0
.. Ip6ReasmReqds                   0                  0.0
.. Ip6ReasmOKs                     0                  0.0
.. Ip6ReasmFails                   0                  0.0


On the transmit side, ``Ip6FragOKs`` counts the number of (large)
packets that were successfully fragmented. ``Ip6FragCreates`` is incremented
for each fragment created. Finally, ``Ip6FragFails`` tracks the failed
fragmentations. If the first two counters increase on a host, it would
be useful to determine which application is sending packets that needs to
be fragmented by the IPv6 stack.
   
.. Ip6FragOKs                      0                  0.0
.. Ip6FragFails                    0                  0.0
.. Ip6FragCreates                  0                  0.0


Multicast and broadcast
-----------------------

The Linux stack also maintains specific packet and byte counters
for the incoming and outgoing multicast and broadcast packets. Their
names are self-describing : ``Ip6InMcastPkts``, ``Ip6OutMcastPkts``,
``Ip6InMcastOctets``, ``Ip6OutMcastOctets``, ``Ip6InBcastOctets``
and ``Ip6OutBcastOctets``. Although the last two counters are defined, they
are never incremented in recent Linux kernels. This should not be a
surprise since there is no broadcast address in IPv6 :rfc:`4291`.


.. Ip6InMcastPkts                  50086              0.0
.. Ip6OutMcastPkts                 40                 0.0
.. Ip6InMcastOctets                3507519            0.0
.. Ip6OutMcastOctets               3760               0.0
.. Ip6InBcastOctets                0                  0.0
.. Ip6OutBcastOctets               0                  0.0


Congestion notification
-----------------------

The last group of IPv6 counters track the usage of Explicit Congestion
Notification (ECN) :rfc:`3168`. ECN uses the two least significant bits
of the Traffic Class field in the IPv6 header. The four counters track
packets with the four possible values:

 - ``00`` : this is a packet sent by a host which does not support ENC.
   ``Ip6InNoECTPkts`` is incremented for each such packet received.
 - ``10`` : this is a packet sent by a host which supports ECN and did not experience congestion. ``Ip6InECT0Pkts`` is incremented for each such packet received.
 - ``01`` : this is a packet using ECT(1). A TCP stack should not send such as packet, but other protocols such as L4S use this code point. ``Ip6InECT1Pkts`` is incremented for each such packet received.
 - ``11`` : this is a packet sent by a host which supports ECN and did experience congestion. ``Ip6InCEPkts`` is incremented for each such packet received.  

ECN is currently not widely deployed, neither on hosts nor on routers. Linux
controls the utilization of ECN with two ``sysctl`` variables:
``net.ipv4.tcp_ecn`` and ``net.ipv4.tcp_ecn_fallback``. The former should be
set to ``1`` to fully support ECN, but the default is ``2`` which indicates
that Linux enables ECN when requested on incoming connections but not on
outgoing ones. The latter is a fallback mechanism which is always enabled
by default. 

   
   
.. Ip6InNoECTPkts                  4574552            0.0
.. Ip6InECT1Pkts                   0                  0.0
.. Ip6InECT0Pkts                   137799             0.0
.. Ip6InCEPkts                     0                  0.0
