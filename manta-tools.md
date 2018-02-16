# Manta Tools

<!--
   Flesh out cut/awk/sort?
   Perf section?
   - Why is it slow?
     - What is it?
       - `svcs -p`
       - pg_stat_activity
     - What's it doing?
       - On CPU? Profile it.
       - Off CPU? Profile where it goes off CPU.
-->



## Background

Manta Operator's Guide:
https://joyent.github.io/manta

- Design goals
- Architecture and components
- Ops procedures: deployment, upgrade, debugging



## This deck (highlights)

* Manta operator tools
  * Taking stock: `manta-adm show`, `manta-adm cn`
  * Getting around: `manta-login`, `manta-oneach`
  * And more: `mlive`, `manta-adm alarm`



## This deck (highlights)

* Lower-level tools:
  * Metadata: `mlocate`
  * Moray: `moray(1)` suite
  * Marlin: `mrjob`, `mrzones`, `mrgroups`
  * PostgreSQL: `pgsqlstat`



## This deck (highlights)

* Tips, Tricks, and Gotchas (h/t DTrace)



# Manta Tools



## Taking stock

- `manta-adm show [SERVICE]`: show zones
- `manta-adm show -s`: count zones by shard, version
- `manta-adm cn`: show CNs



### List zones in local datacenter

    [root@headnode (staging-1) ~]# manta-adm show
    SERVICE          SH ZONENAME                             GZ ADMIN IP     
    authcache         1 51ae8301-fefa-4922-9c48-93226acec1a5 172.25.3.38     
    electric-moray    1 2a689e6c-e12a-4c62-bc7c-72b7efab7eea 172.25.3.38     
    jobpuller         1 9d1ffdcd-a283-4835-8cb0-20c2fa77c134 172.25.3.38     
    jobsupervisor     1 2b1d10be-6d68-4fe8-91e5-688aeae0cae9 172.25.3.38     
    loadbalancer      1 49a3d111-c7a3-478a-9d9c-8ec85a0f64da 172.25.3.38     
    ...
    moray             1 b3c6c144-7500-4904-a0b5-91997a71f75d 172.25.3.38     
    moray             2 b68396db-d49f-487a-8379-a36234ac9993 172.25.3.38     
    moray             3 69790d4a-d500-4e56-99ac-967024765805 172.25.3.38     
    nameservice       1 c6bfb16d-1c43-4fff-95be-5603870dd822 172.25.3.38     
    ...



### Tip: how many shards?

    [root@headnode (staging-1) ~]# manta-adm show postgres
    SERVICE          SH ZONENAME                             GZ ADMIN IP     
    postgres          1 70d44638-f4fb-4cbd-8611-0f7c83d8502f 172.25.3.38     
    postgres          2 a5223321-600b-43eb-b66d-ebc0ab500046 172.25.3.38     
    postgres          3 ef318383-c5fb-4381-8416-a1636d8efa95 172.25.3.38     



### List specific columns

    [root@headnode (staging-1) ~]# manta-adm show \
        -o gz_admin_ip,service,zonename,primary_ip storage
    GZ ADMIN IP      SERVICE          ZONENAME                             PRIMARY IP      
    172.25.3.39      storage          f7954cad-7e23-434f-be98-f077ca7bc4c0 172.27.3.14     

* See `manta-adm help show` for a list of fields you can pass to `-o`.



### List zones in all datacenters within the region

    [root@headnode (staging-1) ~]# manta-adm show -a
    SERVICE          SH DATACENTER ZONENAME                            
    authcache         1 staging-1  51ae8301-fefa-4922-9c48-93226acec1a5
    authcache         1 staging-2  8d01425c-a7d7-4373-9c05-c46b0a567ab9
    authcache         1 staging-3  46ebba2d-820f-4331-ad25-37062192e68c
    electric-moray    1 staging-1  2a689e6c-e12a-4c62-bc7c-72b7efab7eea
    electric-moray    1 staging-2  56533ef1-f79e-4406-9d2a-0b72fb2d2394
    electric-moray    1 staging-3  b6f4ec17-f7ea-4c38-b741-de88e992c5d3
    ...

* Some information is not available about zones in other datacenters (including
  images/versions, IPs, and server info).



### List zones grouped by compute node

    [root@headnode (staging-1) ~]# manta-adm show -c
    CN RA10146    445aab6c-3048-11e3-9816-002590c3f3bc 172.25.3.39     
         SERVICE          SH ZONENAME                            
         marlin            1 0474da9a-21cd-4eba-a0c2-c353a14b2fbd
    ...
         storage           1 f7954cad-7e23-434f-be98-f077ca7bc4c0
    CN RA14872    aac3c402-3047-11e3-b451-002590c57864 172.25.3.38     
         SERVICE          SH ZONENAME                            
         authcache         1 51ae8301-fefa-4922-9c48-93226acec1a5
         electric-moray    1 2a689e6c-e12a-4c62-bc7c-72b7efab7eea
         jobpuller         1 9d1ffdcd-a283-4835-8cb0-20c2fa77c134
         jobsupervisor     1 2b1d10be-6d68-4fe8-91e5-688aeae0cae9
    ...



### List images in use

    [root@headnode (staging-1) ~]# manta-adm show -s
    SERVICE          SH VERSION                                    COUNT
    authcache         1 master-20171201T230631Z-gf5ad52f               1
    electric-moray    1 master-20180118T080706Z-g27c91ec               1
    jobpuller         1 master-20170920T001026Z-g95b5f73               1
    jobsupervisor     1 master-20170919T234226Z-gb8af6a8               1
    loadbalancer      1 master-20180111T211740Z-ga9c40a1               1
    marlin            1 master/13.3.6                                 32
    medusa            1 master-20170919T234916Z-g7b01c12               1
    moray             1 master-20180118T075404Z-g76db9a2               1
    moray             2 master-20180118T075404Z-g76db9a2               1
    moray             3 master-20180118T075404Z-g76db9a2               1
    nameservice       1 master-20171215T001927Z-g5958142               1
    ...



### Tip: Mapping zonenames to storage ids

* Local datacenter:

      [root@headnode (staging-1) ~]# manta-adm show -o zonename,storage_id storage
      ZONENAME                             STORAGE ID                
      f7954cad-7e23-434f-be98-f077ca7bc4c0 1.stor.staging.joyent.us  

* Whole region:

      [root@headnode (staging-1) ~]# manta-adm show -a -o zonename,storage_id storage
      ZONENAME                             STORAGE ID                
      f7954cad-7e23-434f-be98-f077ca7bc4c0 1.stor.staging.joyent.us  
      12fa9eea-ba7a-4d55-abd9-d32c64ae1965 2.stor.staging.joyent.us  
      6dbfb615-b1ac-4f9a-8006-2cb45b87e4cb 3.stor.staging.joyent.us  



### `manta-adm cn`

Show various information about servers.



### List compute nodes

    [root@headnode (staging-1) ~]# manta-adm cn
    DC        HOST              ADMIN IP         KIND   
    staging-1 RA14872           172.25.3.38      other  
    staging-1 RA10146           172.25.3.39      storage

See also: `-o` option for specific fields



### Tip: Mapping CNs to compute ids

    [root@headnode (staging-1) ~]# manta-adm cn -o host,compute_id
    HOST              COMPUTE ID              
    RA14872           -                       
    RA10146           1.cn.staging.joyent.us  



### Tip: Mapping CNs to storage ids

    [root@headnode (staging-1) ~]# manta-adm cn -o host,storage_ids
    HOST              STORAGE IDS               
    RA14872           -                         
    RA10146           1.stor.staging.joyent.us  

* Gotcha: it's `storage_ids`, not `storage_id`.  There may be more than one!
  (Generally only in dev.)



### What's broken

* See `manta-adm alarm`



### `manta-adm alarm`

<!-- XXX manta-adm alarm -->



### Loads more details

    man manta-adm


## `manta-login`

* Log into zone 586053f4-bd49-44bf-bf3e-56436bc0a0ce:

      manta-login 586053f4-bd49-44bf-bf3e-56436bc0a0ce

* Opens a shell in a specific zone (using `ssh` and `zlogin`).
* Tip: Argument can be any substring of the zone uuid.
* Tip: If there's more than one match, you'll get a list to pick from.



### `manta-login SERVICE [WHICH]`

* Show a list of "jobsupervisor" zones from which to pick:

      manta-login jobsupervisor

* Log into the first "postgres" zone (generally part of shard 1):

      manta-login postgres 0



### `manta-login` numbering

* Gotcha: shards numbered from 1, but `manta-login` options numbered from 0:

      [root@headnode (staging-1) ~]# manta-login postgres
      0:   postgres          1 70d44638-f4fb-4cbd-8611-0f7c83d8502f 172.25.3.38     
      1:   postgres          2 a5223321-600b-43eb-b66d-ebc0ab500046 172.25.3.38     
      2:   postgres          3 ef318383-c5fb-4381-8416-a1636d8efa95 172.25.3.38     
      Choose a number: 



### `manta-login` matching

* Gotcha: "moray" matches both "electric-moray" and "moray"

      [root@headnode (staging-1) ~]# manta-login moray
      0:   electric-moray    1 2a689e6c-e12a-4c62-bc7c-72b7efab7eea 172.25.3.38     
      1:   moray             1 b3c6c144-7500-4904-a0b5-91997a71f75d 172.25.3.38     
      2:   moray             2 b68396db-d49f-487a-8379-a36234ac9993 172.25.3.38     
      3:   moray             3 69790d4a-d500-4e56-99ac-967024765805 172.25.3.38     
      Choose a number: 



### `manta-login -G`

* With `-G`, opens a shell in the global zone of the server hosting the matching
  zone.
* Useful for looking at system-wide stats and DTrace probes.

* Log into the GZ of the shard 1 postgres:

      manta-login -G postgres 0



### More details

    man manta-login



## `manta-oneach`

* `manta-login` is great for test systems and making one-off observations and
  tweaks
* In production, `manta-oneach` is needed for _ad hoc_ changes or data
  collection



## Be careful!

**`manta-oneach` is a very sharp tool!**

(See: accidental whole-datacenter reboot.)
<!-- XXX link to postmortem -->



### Example: restart all jobsupervisors

    [root@headnode (emy-10) ~]# manta-oneach -s jobsupervisor \
        'svcadm restart jobsupervisor'
    SERVICE          ZONE     OUTPUT
    jobsupervisor    2a74b4a2 
    jobsupervisor    d8800db3 



### Example: restart all Marlin agents

    [root@headnode (emy-10) ~]# manta-oneach -G -s storage \
        'svcadm restart marlin-agent'
    HOSTNAME              OUTPUT
    headnode              



### Tip: tabular output

* By default: if all results are one line, show a table.
* Force table (using only the last line of output) with `-N`



### Example: sample Muskie logs for recent errors

    [root@headnode (emy-10) ~]#     manta-oneach -s webapi \
        'tail -n 100 /var/log/muskie.log | grep -c "handled: 5"'
    SERVICE          ZONE     OUTPUT
    webapi           1aab0a8f 0
    webapi           2004385c 0



### Example: fetch ZFS storage used

    [root@headnode (staging-1) ~]# manta-oneach -s postgres \
        -N 'zfs list -o used'
    SERVICE          ZONE     OUTPUT
    postgres         70d44638 8.27G
    postgres         a5223321 11.5G
    postgres         ef318383 11.5G



### Bonus tip: skip header row

* Many commands support `-H` to skip the header row
* List status of loadbalancer haproxy instances:

      [root@headnode (emy-10) ~]# manta-oneach -s loadbalancer \
          'svcs -H haproxy'
      SERVICE          ZONE     OUTPUT
      loadbalancer     4409d3c7 online         Nov_25   svc:/manta/haproxy:default
      loadbalancer     d5d07015 online         Nov_19   svc:/manta/haproxy:default



### Bonus tip: select fields:

* Many commands support `-o` to select columns
* More concise than the last one:

      [root@headnode (emy-10) ~]#       manta-oneach -s loadbalancer \
          'svcs -H -o state haproxy'
      SERVICE          ZONE     OUTPUT
      loadbalancer     4409d3c7 online
      loadbalancer     d5d07015 online



### File transfers

* Distribute a tracing script:

      manta-oneach -s postgres -d /root -g $PWD/trace_10s.d
      manta-oneach -s postgres 'chmod +x /root/trace_10s.d'

* Trace for a while (make sure this script exits!):

      manta-oneach -s postgres '/root/trace_10s.d`



### File transfers (2)

* Collect files:

      manta-oneach -s postgres -d $PWD -p /var/tmp/data.out

* Puts one file from each zone into the current directory, named by the zone
  uuid.



### Tip: concurrency

* Default concurrency: up to 30 at a time
* Can adjust this with `--concurrency=N`



### Tip: immediate output

* By default, waits for all results before printing anything.
* Use `-I` to get results immediately.



### Trick: staggered restarts

Restart loadbalancers a few seconds apart:

    manta-oneach -I --concurrency=1 -s loadbalancer \
        'sleep 5; date; svcadm restart haproxy'



### Gotcha: timeouts and processes

* Default: 60-second execution timeout (override with `-T NSECONDS` &mdash;
  especially useful when a server is down).
* **Processes are never cleaned up automatically!**



### Gotcha: timeout leaves process running

<!-- XXX example -->



### Gotcha: processes left running

* Make sure all processes will exit on their own (or at least that you can kill
  them easily with another invocation)



### Tip: long-running scripts

* For long-running tracing scripts, name them with a ticket number (e.g.,
  `MANTA-1234.d`)
* Can use:

      manta-oneach ... 'pkill MANTA-1234.d'



### wanted: `manta-oneach` improvements

* Select only zones for shard N with `--shard`:

      manta-oneach -s postgres --shard=14 'manatee-adm show'

* Select only primaries, syncs, asyncs, etc.:

      manta-oneach --manatee-role=primary \
          'pgrep -U postgres | wc -l'

* Option for `-G` that invokes the command once for each matching instance,
  providing zonename &mdash; e.g.,

      manta-oneach -G --for-each \
          -s storage 'vmadm get $ZONENAME`



### wanted: `manta-login` improvements

* Same flags as `manta-oneach` has: `--service`
* The goodies on the previous slide



### More details

    man manta-login



## `mlive`

* Very primitive load generator
* ("is it up?")
* https://github.com/joyent/manta-mlive
* Download and run from SmartOS zone (or Macbook, for flaky results)



### First time for a region

* Use `-s N` to specify number of shards (used &mdash; dubiously &mdash; to try
  to create enough different directories to hit every shard)

<!-- XXX `mlive` output -->



### Subsequent times

* Use `-S` to skip re-creating the whole directory tree
* If you use this when you haven't created it before (or created it with a
  different number of shards), you'll see false positives!

<!-- XXX `mlive output` -->



### Broken

<!-- XXX `mlive output`, broken -->



# Internal debugging tools



## `mlocate`

* Shows object metadata
  * Creator, owner, size, user headers
  * Which metadata shard it's on
  * Which sharks store copies of the object



### `mlocate` output

<!-- mlocate output, labeled -->




## `mrjob`

<!-- XXX example -->



## `moray tools`

<!-- XXX flesh out -->



## `pgsqlstat`

<!-- XXX flesh out -->



## `moraystat.d`

<!-- XXX flesh out -->



# OS Tools

* `svcs(1)` (SMF services)
* `proc(1)` ("ptools")
* `netstat(1)` (net connections)
* `netstat(1)` -s (net stats)
* `zonememstat(1M)` (memory limits)
* `vfsstat(1M)` (filesystem ops)
* `prstat(1M)` (process activity)
* `mpstat(1M)` (CPU activity)
* `iostat(1M)` (disk activity)
* `dtrace(1M)`
* `mdb(1M)`, `mdb_v8`



# Worth learning

* `grep -c` to just report a count
* `cut` or `awk` syntax to pull out specific fields
* `sort` options (sort by various columns, numeric sort, `--debug` on GNU)
* `sort | uniq -c`: frequency count, sorted by value
* `sort | uniq -c | sort -n`: frequency count, sorted by count
* OS X: `pbcopy`, `pbpaste`
* `json(1)`, `bunyan(1)`



# Next steps

- Set up your own Manta
- Poke around with these tools
- Watch it with mlive
- Break it!
