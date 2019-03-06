# Q&A

#### Q: hbase:meta 数据丢失

HBase keep having region in transition: 

Regions in Transition

Region State RIT time (ms)

| 1588230740                                                   | hbase:meta,,1.1588230740 state=FAILED_OPEN, ts=Thu Apr 23 12:15:49 ICT 2015 (8924s ago), server=02slave.mabu.com,60020,1429765579823 | 8924009 |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------- |
| Total number of Regions in Transition for more than 60000 milliseconds | 1                                                            |         |
| Total number of Regions in Transition                        | 1                                                            |         |

解决方案：

1. Stop HBase
2. Move your original /hbase back into place
3. Use a zookeeper cli such as "hbase zkcli"[1] and run "rmr /hbase" to delete the HBase znodes
4. Restart HBase. It will recreate the znodes

If Hbase fails to start after this, you can always try the offline Meta repair:
hbase org.apache.hadoop.hbase.util.hbck.OfflineMetaRepair

Also check for inconsistencies after HBase is up.  As the hbase user, run "hbase hbck -details". If there are inconsistencies reported, normally I would use the "ERROR" messages from the hbck output to decide on the best repair method, but since you were willing to start over just run "hbase hbck -repair".

If the above fails, you can always try the offline Meta repair:

hbase org.apache.hadoop.hbase.util.hbck.OfflineMetaRepair

[1] <http://hbase.apache.org/book.html#trouble.tools>
[2] <http://hbase.apache.org/book.html#hbck.in.depth>

