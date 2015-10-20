# PerfAdvisor (PERFormance adVISOR)

Reference : https://wiki.freebsd.org/IdeasPage#PerfVisor_.28PERFormance_adVISOR.29

The goal of this project is to:
- build a tool which can analyze a system for performance bottlenecks and maybe even give some hints what to do next.

Additional References to get us going:

- http://dtrace.org/blogs/brendan/2012/12/13/usenix-lisa-2012-performance-analysis-methodology/
- http://queue.acm.org/detail.cfm?id=2413037 or http://dtrace.org/blogs/brendan/2012/02/29/the-use-method/
- http://dtrace.org/blogs/brendan/2012/03/01/the-use-method-solaris-performance-checklist/
- http://dtrace.org/blogs/brendan/2012/03/07/the-use-method-linux-performance-checklist/
- http://www.slideshare.net/brendangregg/zfsperftools2012 or http://www.brendangregg.com/Slides/zfsperftools2012.pdf
- http://www.dtracebook.com
- http://queue.acm.org/detail.cfm?id=1809426
- http://dtrace.org/blogs/brendan/2012/12/10/usenix-lisa-2010-visualizations-for-performance-analysis/
- http://dtrace.org/blogs/brendan/2012/03/26/subsecond-offset-heat-maps/
- and http://dtrace.org/blogs/brendan/ in general...

Project phases

Initial phase

Identify/list possible resources in FreeBSD. 
See the performance checklist postings above for ideas. 

Those are just a start, this can iteratively be extended to include subitems (e.g. in a first iteration the "network stack" could be an item of a resource, in a second iteration each network interface is a resource, on a third iteration one network protocol could be a resource, on a fourth iteration just the TX queue of a NIC can be a resource, ...). Not everything needs to be identified initially, at one point the work on this resource list should be pushed back to a point where all or some of the following items are handled for the existing resources. 

When all OS resources are listed, further iterations could even include 3rd party applications (e.g. apache httpd requests, mysql table scans, ...). Interesting values for each resource can be bandwidth (total/percentage), operations per second, latency, usage duration, time until completion, ... Ideally the result is written down in the FreeBSD wiki. For each "high-level" resource (e.g. network stack) try to document some kind of "where to look next if saturated/on-error/..." resource (e.g. network interface, protocol checks, ...)
For each resource identify/document "utilization", "saturation" and "error" values (see the USE method in the prerequisites), and code points in the source which show them. If there's no code point in the source, mark the resource for later investigation. Hardware provided info, e.g. CPU counters, count as as "code points" in this sense too.
For each resource determine/document existing tools which provide the required information (saturation/error) and how to use them to get the required info.
For each resource without an existing way to get the required information, but with a code point in the source which provides the required information, write a dtrace/hwpmc/whatever script/program/whatever to query the required information. <- Here is where our health check script can be reused. Abhishek & I have already started the foundations. we are at very very early stage.

For each resource without a code point in the source which provides the required information, find a "cheap" way to determine the resource value and add corresponding code readable by sysctl or dtrace (readout variable on existing probe or a specific SDT probe just for this) or whatever (and write a script/program/whatever as above). <- Payes is doing this stuff. 

Improvement phase

For each existing code point with an existing script check if a new way of handling this info (sysctl, dtrace probe, ...) would make the sampling of this data cheaper (= less performance impact of the performance monitoring itself).
Visualization phase <- this is the tuning aspect, we shall do this when things are in place.

Write a tool (for the ports collection) which visualizes data (heat maps, flame graphs, ...) gathered from the above performance data collection scripts (either stored somewhere or by calling the scripts directly) and provides hints where to look next in case a resource is the bottleneck. The blog of Brendan Gregg gives a lot of ideas how to visualize with various types of graphs. Suggestion: make it system independent, foresee use on *BSD, Solaris and Linux. 

Cloud phase (everything is cloudy nowadays...)

Write an agent which is able to collect the data which the visualizing tool is able to handle and extend the visualizing tool to collect data from agents on other machines (via an encryted and authenticated connection if needed = possible but not mandatory, depending on the environment).
Datawarehouse phase (buzzword bingo!) <- userspace to userspace, kernel to userspace & finally. machine to machine communication. Payes & I will start thinking on this. devd refactoring or something new altogether.

Write a gateway/collector service which a) is able to "handle" agents in a datacenter (query and store data), b) is able to handle bulk-data-retrieval requests from remote locations (transfer data from remote datacenters to the company headquater), and c) is able to store/archive queried data for later use ("How did this look last week/month/year?").

Discussion

The first item in the initial phase is a big item. It may be better to see the complete initial phase as an iterative or divide and conquer step. First determine some high level resources and process all steps of the initial phase on them. Then repeat the initial phase again by braking up the high level resources in more detailed items (e.g. first "all CPU in system" as one resource, second "each CPU in system" as a resrouce, third breaking up each CPU via hardware performance counters).
