---
stand_alone: true
ipr: trust200902
cat: info
submissiontype: IETF
area: Operations and Management
wg: BPM

docname: draft-liu-bpm-aspa-verification-benchmarking-00

title: Benchmarking Methodology for ASPA Verification
abbrev: ASPA Benchmarking
lang: en

author:
- ins: L. Liu
  name: Libin Liu
  org: Zhongguancun Laboratory
  city: Beijing
  country: China
  email: liulb@zgclab.edu.cn

normative:
  RFC2119:
  RFC8174:
  RFC4271:
  RFC6793:
  RFC9234:
  RFC1242:
  RFC4098:  
  I-D.ietf-sidrops-aspa-profile:
  I-D.ietf-sidrops-aspa-verification:
  I-D.ietf-sidrops-8210bis:
 
informative:
  RFC6480:
  RFC6811:
  RFC6815:
  I-D.lg-bmwg-benchmarking-methodology-for-rov:

...

--- abstract

Autonomous System Provider Authorization (ASPA) provides a Resource Public Key Infrastructure (RPKI)-based mechanism for verifying the Border Gateway Protocol (BGP) AS_PATH attribute to detect route leaks and certain forms of AS_PATH manipulation. The performance of ASPA-based AS_PATH verification depends on several factors, including the size and structure of ASPA data, AS_PATH characteristics, verification procedures, and the interaction between ASPA updates and BGP routing operations.
This document defines a benchmarking methodology for evaluating ASPA verification implementations on routing devices. The methodology specifies controlled test conditions and procedures for measuring ASPA data processing, AS_PATH verification performance, route revalidation, BGP convergence, scalability, and resource utilization. The objective is to provide repeatable and comparable measurements that characterize the performance and operational behavior of ASPA verification under varying conditions.

--- middle

# Introduction

The Border Gateway Protocol (BGP) {{RFC4271}} is the fundamental inter-domain routing protocol of the Internet. However, BGP does not inherently provide mechanisms to verify the authenticity of routing information, making it vulnerable to routing anomalies such as route hijacks and route leaks. The Resource Public Key Infrastructure (RPKI) {{RFC6480}} provides a framework for improving inter-domain routing security. Route Origin Validation (ROV) {{RFC6811}} enables routers to verify whether the origin Autonomous System (AS) of a BGP route is authorized to announce the corresponding prefix. However, ROV does not validate the AS_PATH attribute and therefore cannot detect routing anomalies that preserve a legitimate origin AS.

Autonomous System Provider Authorization (ASPA) {{I-D.ietf-sidrops-aspa-profile}} extends RPKI-based routing security by allowing an AS to authorize its provider ASes. Based on validated ASPA information, the AS_PATH verification procedures defined in {{I-D.ietf-sidrops-aspa-verification}} enable routers to detect route leaks and certain forms of AS_PATH manipulation. The verification procedures examine provider authorization relationships along the AS_PATH and produce one of three results: Valid, Invalid, or Unknown. The applicable verification procedure depends on the relationship between the receiving AS and its BGP neighbor.

The introduction of ASPA verification may impose additional processing and resource requirements on routing devices. Unlike ROV, which primarily evaluates prefix-origin authorization, ASPA verification examines the relationships between ASes along an AS_PATH. Its performance may therefore be affected by ASPA-specific characteristics, including AS_PATH length, ASPA coverage, provider-set cardinality, and verification direction. Furthermore, routers need to process ASPA information received through the RPKI-to-Router (RTR) protocol {{I-D.ietf-sidrops-8210bis}}. Changes in ASPA information may also trigger the revalidation of previously received BGP routes, potentially affecting BGP control-plane processing and convergence.

As ASPA verification is implemented in routing devices, a consistent and repeatable benchmarking methodology is needed to characterize its performance under controlled conditions. Such a methodology should enable network operators, equipment vendors, and researchers to evaluate the processing overhead of ASPA verification, understand its scalability limitations, and compare implementation performance. In particular, it is important to distinguish the performance impact of BGP-triggered AS_PATH verification from that of ASPA-triggered route revalidation, as these operations involve different processing conditions and workload characteristics.

Existing work has defined a benchmarking methodology for ROV {{I-D.lg-bmwg-benchmarking-methodology-for-rov}}. Although ROV and ASPA verification share certain operational components, their validation inputs, procedures, and performance characteristics differ. Therefore, this document defines an independent benchmarking methodology specifically for ASPA-based AS_PATH verification, while following established BMWG benchmarking principles.

This document specifies a set of metrics, test conditions, and procedures for evaluating ASPA data processing, AS_PATH verification performance, route revalidation, BGP convergence, scalability, and resource utilization. The methodology uses controlled ASPA datasets and BGP workloads in isolated laboratory environments to support repeatable and comparable measurements across different routing implementations. 
The resulting benchmarks are intended to characterize the performance and operational behavior of ASPA verification and provide a basis for evaluating its deployment on routing devices.

## Requirements Language

{::boilerplate bcp14-tagged}

# Scope and Goals

This document specifies a laboratory-based benchmarking methodology for evaluating the performance of routing devices implementing ASPA-based AS_PATH verification as defined in {{I-D.ietf-sidrops-aspa-verification}}. 
The methodology focuses on the processing of ASPA information, AS_PATH verification, and their impact on BGP control-plane operations under controlled and reproducible conditions.

The scope of this benchmarking methodology includes:  
- ASPA Data Processing Performance: Measurement of the time and resources required for a router to process ASPA information received through the RPKI-to-Router (RTR) protocol, including initial synchronization, incremental updates, and recovery operations.  
- AS_PATH Verification Performance: Evaluation of verification latency and processing throughput under varying AS_PATH characteristics, ASPA coverage levels, provider-set sizes, verification directions, and verification outcomes.  
- ASPA-triggered Route Revalidation: Measurement of the time and resources required to re-evaluate previously received BGP routes following changes in ASPA information, considering the number of affected routes and distinct AS_PATHs.  
- Impact on BGP Control-plane Performance: Quantification of how enabling ASPA verification affects BGP route processing and convergence, including the impact of ASPA updates on routing decisions.  
- Scalability under Controlled Conditions: Evaluation of a router's ability to handle increasing ASPA dataset sizes, provider authorization relationships, BGP route populations, and concurrent ASPA and BGP update workloads.  
- Resource Utilization: Measurement of CPU utilization, memory consumption, and relevant control-plane processing overhead associated with ASPA verification under different workload conditions.  

The goals of this document are:  
- To establish a consistent and repeatable methodology for benchmarking ASPA verification implementations under controlled laboratory conditions.  
- To define metrics and test procedures that enable meaningful performance comparisons across different routing devices and implementations.  
- To characterize the performance impact and scalability of ASPA verification under varying ASPA datasets, AS_PATH characteristics, and BGP workloads.  
- To provide quantitative information that helps network operators and equipment vendors evaluate the operational performance of ASPA verification implementations and make informed deployment decisions.  

The methodology focuses on ASPA verification performed by routing devices, primarily for IPv4 and IPv6 unicast routes received over external BGP (eBGP) sessions, consistent with {{I-D.ietf-sidrops-aspa-verification}}. The benchmarking procedures use controlled ASPA datasets and BGP workloads to isolate the performance effects of ASPA verification from external dependencies.

This document does not benchmark the generation, publication, or cryptographic validation of ASPA objects within the RPKI infrastructure. It also does not define new ASPA verification algorithms, evaluate the security effectiveness of ASPA against routing attacks, or prescribe routing policies based on verification results. While expected verification outcomes are used to establish valid test conditions, protocol conformance testing is outside the scope of this document.

The benchmarking procedures are intended for isolated laboratory environments and are not intended to be conducted on production networks carrying live traffic.

# Terminology

The terminology used in this document follows the conventions of {{RFC1242}}, {{RFC4098}}, and subsequent BMWG publications. Terms related to ASPA objects, provider authorization, and AS_PATH verification are defined in {{I-D.ietf-sidrops-aspa-profile}} and {{I-D.ietf-sidrops-aspa-verification}}.

In particular, this document adopts the definitions of Customer AS (CAS), Provider AS (PAS), Set of Provider ASes (SPAS), Union SPAS (U-SPAS), and the provider authorization results Provider+, Not Provider+, and No Attestation. The upstream and downstream verification procedures, as well as the verification results Valid, Invalid, and Unknown, are also defined in {{I-D.ietf-sidrops-aspa-verification}}.

The following terms are used with specific meanings in the context of ASPA benchmarking.

Device Under Test (DUT): A routing device or software router implementing ASPA-based AS_PATH verification whose performance is evaluated using the methodology defined in this document.

ASPA Dataset: A collection of ASPA-derived customer-to-provider authorization information provided to the DUT through the RPKI-to-Router (RTR) protocol. Each active CAS entry is associated with a provider set. The dataset represents the information used by the DUT for ASPA verification rather than the underlying RPKI signed objects.

ASPA Record Count: The number of distinct CAS entries in an ASPA dataset. Each CAS entry is counted once, regardless of the number of authorized providers associated with it.

Provider Authorization Count: The total number of distinct CAS-to-PAS authorization relationships represented in an ASPA dataset. The special AS 0 value, which indicates the absence of transit providers, is not counted as a provider authorization.

Provider-Set Cardinality: The number of authorized Provider ASes associated with a particular CAS. For benchmarking purposes, provider-set cardinality may be characterized by its minimum, maximum, mean, and distribution across an ASPA dataset.

ASPA Coverage: The proportion of distinct ASes in a specified reference AS population for which ASPA information is available. For benchmarking, the reference population is derived from the ASes relevant to provider authorization checks in the test workload. The definition and calculation of ASPA coverage are specified in Section 5.

AS_PATH Length: The number of ASNs in an AS_PATH. This document distinguishes between raw AS_PATH length and compressed AS_PATH length, where consecutive duplicate ASNs are removed according to {{I-D.ietf-sidrops-aspa-verification}}. Unless otherwise specified, AS_PATH length in benchmarking tests refers to compressed AS_PATH length.

Unique AS_PATH Count: The number of distinct AS_PATH sequences represented in a BGP route workload. Multiple routes sharing the same AS_PATH are counted as one unique AS_PATH. The benchmark specifies whether uniqueness is determined using raw or compressed AS_PATHs.

Verification Outcome Distribution: The proportion of BGP routes in a test workload expected to produce each ASPA verification result: Valid, Invalid, or Unknown.

ASPA Update Transaction: A controlled set of ASPA record additions, replacements, or withdrawals delivered to the DUT through the RTR protocol as part of a single cache response transaction.

ASPA Churn: The rate and pattern of changes to ASPA information, including additions, replacements, and withdrawals of CAS entries, during a specified measurement interval.

Affected Route Population: The set of previously received BGP routes whose expected ASPA verification results change as a consequence of a specified ASPA dataset update. The population is determined from the test workload and the expected verification results before and after the update.

Route Revalidation: The re-evaluation of previously received BGP routes following changes in ASPA information that may affect their verification results.

Steady State: A test condition in which the ASPA dataset, BGP route population, and relevant routing states have stabilized according to predefined criteria, allowing performance measurements to begin under controlled conditions.

# Test Setup and Laboratory Environment {#test_setup}

This section describes the laboratory environment for benchmarking ASPA-based AS_PATH verification implementations. 
The test setup provides controlled ASPA information and BGP routing workloads to the Device Under Test (DUT), while supporting measurements of ASPA processing, route verification, BGP control-plane behavior, and resource utilization.

The test environment is designed to isolate the performance of ASPA verification from external dependencies and to ensure that test conditions can be reproduced across different implementations.  

## Test Topology

~~~~~~~~~~
+---------------------+      RTR
| ASPA Data Generator |------------+
|   (RTR Emulator)    |            |
+---------------------+            v
                            +----------------+
+---------------------+     |                |
| BGP Route Generator |---->|      DUT       |
+---------------------+ BGP |                |
                            +--------+-------+
                                     |
                                     | Monitoring
                                     v
                            +----------------+
                            |  Measurement   |
                            |     System     |
                            +----------------+
~~~~~~~~~~
{: #test-topo title="The test topology for ASPA benchmarking."}

{{test-topo}} illustrates the reference topology for ASPA verification benchmarking. The reference topology consists of the following components:

- Device Under Test (DUT): A routing device or software router implementing ASPA-based AS_PATH verification.  
- ASPA Data Generator: A component that constructs controlled ASPA datasets and update sequences with specified customer-provider authorization relationships.  
- RTR Cache/Emulator: A cache implementation or emulator that delivers ASPA information to the DUT using the RPKI-to-Router (RTR) protocol.  
- BGP Route Generator: A component that establishes BGP sessions with the DUT and generates routes with controlled AS_PATH attributes.  
- Measurement System: A component that collects timing information, verification results, routing-state changes, and resource-utilization measurements.  
- Routing-State Observer: An optional component that monitors externally observable BGP routing behavior through a monitoring session, telemetry interface, or other suitable mechanism.  

The ASPA Data Generator and RTR Emulator MAY be implemented as a single component. 
The test topology MAY include multiple BGP peers to evaluate different neighbor relationships or concurrent routing workloads. The BGP Route Generator can work for this function.  
A Tester MAY also be added to generate and receive data-plane traffic when forwarding behavior is included in a convergence measurement.
The laboratory environment MUST be isolated from production routing infrastructure. Synthetic ASPA information and BGP routes MUST NOT be propagated to operational networks.
The test report MUST describe the topology, the connections between test components, and any additional components used during the measurements.

## DUT Configuration Requirements

The DUT MUST implement the ASPA-based AS_PATH verification procedures specified in {{I-D.ietf-sidrops-aspa-verification}} and support receiving ASPA information through the applicable RTR protocol.  

ASPA verification MUST be enabled for tests that evaluate its processing behavior. Tests comparing ASPA-enabled and ASPA-disabled configurations MUST maintain equivalent BGP workloads and other relevant configurations.  

The DUT configuration MUST specify the relationship between the local AS and each BGP neighbor used in the tests. These relationships determine whether upstream or downstream AS_PATH verification is applied. The following configuration information MUST be documented:  
- DUT hardware platform and software version.   
- Available CPU and memory resources.  
- ASPA verification configuration.  
- RTR protocol version and relevant session parameters.  
- BGP neighbor ASNs and locally configured relationships.  
- BGP address families and session configurations.  
- Routing policies associated with ASPA verification results.  
- Other enabled routing security mechanisms that may affect the measurements.  

The BGP Roles defined in {{RFC9234}} SHOULD be configured consistently with the intended neighbor relationships when supported by the DUT and test peers.  

Routing policies associated with ASPA verification results MUST remain consistent across test runs unless the policy itself is an explicit test variable.  

In particular, the test report MUST indicate whether routes classified as Invalid are made ineligible for route selection. Such policies may affect BGP convergence measurements but do not necessarily change the execution of the ASPA verification procedure.  

When other routing security mechanisms, such as ROV, are enabled, their configurations and associated validation datasets MUST remain unchanged during tests intended to isolate ASPA verification performance. Their presence MUST be reported.  

Non-essential control-plane processes and services SHOULD be disabled or configured consistently to reduce measurement interference. Any logging or telemetry mechanisms used for measurement MUST be documented because they may introduce additional processing overhead.

## ASPA Data Source and RTR Emulation

The test environment MUST provide an RTR data source capable of delivering controlled ASPA datasets and updates to the DUT.  
The data source MAY use an isolated RPKI cache implementation or a dedicated RTR emulator.  
A dedicated emulator can be used to generate ASPA payloads directly without requiring the creation and cryptographic validation of corresponding RPKI signed objects.  
The RTR data source MUST support the ASPA PDU format and processing semantics defined in {{I-D.ietf-sidrops-8210bis}}.  

The ASPA datasets used in benchmarking MUST have reproducible characteristics, including:  
- The number of distinct CAS entries.   
- The total number of CAS-to-PAS authorizations.     
- The provider-set cardinality distribution.  
- The availability of ASPA information for ASes appearing in the test workloads.   
- The authorization relationships required to produce the expected AS_PATH verification results.  

The data source MUST support initial ASPA dataset synchronization and controlled incremental updates.
Incremental updates MUST include the ability to announce new ASPA records, replace existing provider sets, and withdraw existing ASPA records.  
An ASPA announcement or replacement MUST contain the complete provider set for the corresponding CAS, consistent with {{I-D.ietf-sidrops-8210bis}}. Partial modifications of individual providers within a CAS entry MUST NOT be represented as independent incremental ASPA operations.
For an ASPA withdrawal, the data source MUST remove the corresponding CAS entry according to the applicable RTR protocol semantics.  

The data source SHOULD allow the tester to control:  
- The number of CAS entries changed in an update transaction.  
- The number of provider authorizations affected by an update.  
- The timing and frequency of ASPA update transactions.  
- The distribution of updates across different CAS entries.  
- The timing of RTR notifications and responses.  
- RTR session interruption, reconnection, and cache reset events.  

For update-processing measurements, the RTR source SHOULD support repeatable timing of update transactions and SHOULD record timestamps for relevant protocol events, including ASPA payload transmission and End of Data PDUs.  
The completion of an RTR data transaction does not necessarily indicate that the DUT has completed all internal processing associated with the updated ASPA information. 
Therefore, the benchmark MUST distinguish RTR transfer completion from ASPA data application and subsequent route revalidation when these events are measured.  
The ASPA Data Generator SHOULD support both synthetic datasets with controlled characteristics and datasets derived from operational ASPA information. 
If operational data is used, its source, collection time, and any modifications MUST be documented.

Synthetic datasets MUST remain consistent with the ASPA data model and the applicable RTR protocol requirements.
The RTR source MUST be capable of delivering the specified update workload without becoming the performance bottleneck. Its maximum sustainable update generation rate SHOULD be established before conducting DUT scalability or stress tests.

## BGP Traffic Generation Requirements

The BGP Route Generator MUST be capable of establishing one or more eBGP sessions with the DUT and advertising routes with controlled AS_PATH attributes.  
Unlike ROV benchmarking, which primarily controls route prefixes and origin ASes, ASPA benchmarking requires the generation of AS_PATHs with specified structures and provider authorization relationships.  

The BGP Route Generator MUST support configuration of the following parameters:
- The number of advertised BGP routes.  
- The number of distinct AS_PATHs.  
- Raw and compressed AS_PATH lengths.  
- The distribution of routes across distinct AS_PATHs.  
- The expected ASPA verification results.  
- The rate of BGP route announcements and withdrawals.  
- The timing and sequence of BGP updates.  

The generator MUST be capable of constructing AS_PATHs that produce Valid, Invalid, and Unknown verification results under a specified ASPA dataset, according to {{I-D.ietf-sidrops-aspa-verification}}.  
The expected verification result of each test route MUST be determined before the corresponding test begins. 
These expected results SHOULD be generated or independently checked using the verification procedures defined in {{I-D.ietf-sidrops-aspa-verification}}.
The BGP Route Generator MUST also support testing of both upstream and downstream verification procedures by establishing sessions with the appropriate locally configured neighbor relationships.
Generated AS_PATHs MUST satisfy the applicable prerequisite AS_PATH checks unless a test explicitly targets those checks. In particular, the most recently added AS in the AS_PATH MUST match the advertising neighbor's ASN, except for the applicable route-server cases.
The generator SHOULD support four-octet ASNs and construct paths consistent with the AS_PATH handling procedures specified in {{RFC6793}}.

For tests involving AS_PATH prepending, the generator MUST distinguish the raw AS_PATH from the compressed AS_PATH used by the ASPA verification procedures.
The generator SHOULD support multiple BGP routes sharing the same AS_PATH, as well as routes with distinct AS_PATHs. 
These two workload characteristics MUST be independently controlled or documented because an implementation may reuse verification results or internal path representations across multiple routes.

The generator SHOULD support different BGP update patterns, including:
- Individual route announcements and withdrawals.  
- Sequential BGP updates at a controlled rate.  
- Bursts of BGP updates.  
- Updates involving multiple distinct AS_PATHs.  
- Updates affecting routes that share the same AS_PATH.  

The BGP Route Generator MUST be capable of establishing a stable baseline routing table before a benchmark begins.
For tests involving ASPA-triggered route revalidation, the generator MUST maintain the baseline BGP routes without modification while the ASPA dataset is updated.
The generator SHOULD also support synchronized or controlled overlapping BGP and ASPA update sequences for tests involving concurrent control-plane activity.
The generated workloads MAY include IPv4 and IPv6 unicast routes. The number of routes for each address family MUST be reported separately.
The generator MUST be capable of sustaining the specified BGP update rate without becoming the performance bottleneck. Its generation capacity SHOULD be verified independently before testing the DUT.

## Measurement and Instrumentation

The Measurement System collects timing information, routing-state changes, verification results, and resource-utilization measurements during the benchmark tests.  
The measurement setup MUST provide observation points sufficient to evaluate the metrics specified by each benchmark.
Depending on the test, measurements may be collected through:  
- Packet captures of RTR and BGP sessions.  
- Routing-state information exposed by the DUT.  
- Device telemetry or management interfaces.  
- BGP monitoring sessions.  
- CPU and memory utilization counters.  
- Implementation-specific instrumentation, where available.  

The measurement method and observation points MUST be documented for each benchmark.

For timing measurements, the Measurement System SHOULD use a common timestamp source or synchronized clocks with sufficient accuracy for the expected measurement intervals.
When measurements are collected from different devices, clock synchronization accuracy and timestamp resolution MUST be considered when calculating elapsed times.
The benchmarking methodology distinguishes between internal processing measurements and externally observable measurements.

**Internal Processing Measurements**: These measurements characterize operations within the DUT, such as the execution time of ASPA verification or the application of updated ASPA information. They generally require suitable instrumentation provided by the implementation.

**Externally Observable Measurements**: These measurements characterize intervals between observable protocol or routing-state events, such as the receipt of a BGP UPDATE and a subsequent routing-state change.

Externally observable measurements may include additional processing delays beyond ASPA verification itself, including BGP UPDATE parsing, routing policy evaluation, route selection, and message generation.
Therefore, internal ASPA verification latency and externally observable route-processing latency MUST NOT be reported as equivalent measurements.

Where direct ASPA verification results are available through device interfaces, the Measurement System SHOULD collect them to determine when the expected verification states have been reached.
When direct verification-state information is unavailable, the tester MAY use a controlled routing policy to produce observable routing-state changes associated with specified ASPA verification results.
Such observations MUST be identified as routing-state measurements rather than direct measurements of the ASPA verification algorithm.
If the required verification-state transition cannot be observed using the available interfaces, the corresponding verification latency MUST be reported as unavailable rather than inferred from an unrelated event.

For resource-utilization measurements, the Measurement System SHOULD record CPU and memory usage at a consistent sampling interval throughout each test.
The measurement report MUST identify the sampling interval, measurement source, and whether the values represent total system utilization or utilization of specific processes.
Measurement mechanisms SHOULD minimize additional processing overhead on the DUT. Any instrumentation that may affect benchmark results MUST be documented.

## Test Initialization and Steady-State Conditions

Each benchmark MUST begin from a predefined initial state to ensure that measurements are repeatable.  

Unless otherwise specified by an individual benchmark, the following initialization procedure SHOULD be used:

1. Configure the DUT, RTR data source, and BGP Route Generator according to the selected test parameters.  
2. Establish the RTR session and load the baseline ASPA dataset.  
3. Confirm that the RTR synchronization process has completed and that the expected ASPA information is available to the DUT.  
4. Establish the required BGP sessions and advertise the baseline route population.  
5. Confirm that the expected ASPA verification results and routing states have been reached.  
6. Allow the DUT to reach steady state before initiating the test event.  
7. Record baseline CPU and memory utilization and other relevant control-plane measurements.  
8. Initiate the benchmark-specific test procedure.  

The initialization sequence MAY differ for tests that explicitly evaluate initial ASPA synchronization, cold-start behavior, or RTR recovery.  
For steady-state tests, the following conditions SHOULD be satisfied:  
- The baseline ASPA dataset has been synchronized.  
- The required BGP sessions are established.  
- The baseline BGP route population is stable.  
- The expected ASPA verification and routing states have been reached.  
- No unrelated BGP or RTR synchronization activity is in progress.  
- Relevant resource-utilization measurements have stabilized according to predefined criteria.  

The tester MUST define the steady-state criteria before beginning a benchmark.
A benchmark MUST NOT begin if the required initial conditions have not been satisfied.
For repeated measurements, the initial ASPA dataset, BGP route population, and relevant DUT configuration MUST be restored between runs unless a benchmark explicitly evaluates cumulative processing or sustained workload behavior.
The benchmark report MUST identify any deviations from the specified initial conditions.

# Benchmarking Methodology

This section defines the methodology for benchmarking ASPA-based AS_PATH verification on routing devices. It specifies the test conditions, workload characteristics, and measurement metrics required to obtain repeatable and comparable results.

Unlike route origin validation, ASPA-based AS_PATH verification examines provider authorization relationships along an AS_PATH. Its performance may depend on the characteristics of both the ASPA dataset and the BGP routing workload. Therefore, benchmarking requires coordinated control of these inputs to isolate the performance impact of ASPA verification under different conditions.

The methodology defined in this section applies to the benchmark tests specified in Section 6.

## General Considerations

All benchmark tests MUST be conducted under controlled and repeatable conditions using the laboratory environment described in {{test_setup}}.

Unless otherwise specified, the DUT MUST reach a predefined steady state before measurements begin. The initial ASPA dataset, BGP route population, verification configuration, and relevant routing policies MUST be documented.
Each benchmark MUST identify the primary variable being evaluated while keeping other relevant parameters unchanged where feasible. When multiple parameters are varied simultaneously, their values and relationships MUST be explicitly reported.
For tests involving ASPA verification results, the expected results MUST be determined independently of the DUT before measurements begin. These results MUST be consistent with the verification procedures specified in {{I-D.ietf-sidrops-aspa-verification}}.

The observed verification results SHOULD be checked against the expected results to ensure that measurements are collected under valid test conditions. Tests in which the DUT does not reach the expected verification or routing state MUST be identified and MUST NOT be reported as successful benchmark results.
Benchmarking SHOULD include both isolated workloads, which vary individual ASPA-specific characteristics, and combined workloads, which evaluate performance under multiple concurrent conditions.

Where applicable, measurements SHOULD be repeated with ASPA verification disabled to characterize the additional processing overhead associated with enabling ASPA verification. Such baseline comparisons MUST use equivalent BGP workloads and routing configurations, except for the ASPA-related configuration under evaluation.
Each test condition SHOULD be repeated at least ten times. The number of repetitions, measurement variability, and any unsuccessful runs MUST be reported.
Measurement results SHOULD include appropriate summary statistics, such as the mean, median, standard deviation, and selected percentiles where sufficient samples are available.

## Test Control and Input Conditions

The performance of ASPA verification is influenced by the characteristics of the ASPA dataset, the structure of BGP AS_PATHs, and the interaction between ASPA information and BGP routing operations.  

This section defines the primary input parameters that MUST be controlled or documented, as applicable, during benchmarking.  

### ASPA Dataset Size and Structure

The ASPA dataset provided to the DUT is characterized by the number of CAS entries and the associated provider authorizations.  

At minimum, the following parameters MUST be reported:  
- ASPA Record Count: The number of distinct CAS entries in the ASPA dataset.  
- Provider Authorization Count: The total number of distinct CAS-to-PAS authorization relationships.  
- Provider-Set Cardinality: The number of authorized providers associated with each CAS.  

These parameters MUST be distinguished because ASPA datasets containing the same number of CAS entries may differ significantly in the number and distribution of provider authorizations.
For a dataset containing M CAS entries, the provider authorization count can be expressed as:

    P = sum(|S_i|), for i = 1, ..., M

where S_i denotes the set of distinct authorized provider ASes for CAS i, excluding the special AS 0 value.

An ASPA entry containing only AS 0 represents an explicit statement that the CAS has no transit providers, rather than an authorization of AS 0 as a provider. Such entries MUST be distinguished from CAS entries for which no ASPA information is available.

When varying ASPA dataset size, the tester SHOULD distinguish between increasing the number of CAS entries and increasing the number of provider authorizations per CAS.  

The provider-set cardinality distribution SHOULD be documented using statistical characteristics such as minimum, maximum, mean, and median values.  

For tests intended to isolate the effect of provider-set cardinality, the number of CAS entries and other relevant workload characteristics SHOULD remain fixed.  

The tester MAY use datasets derived from operational ASPA information or synthetically generated datasets. Synthetic datasets SHOULD be used when precise control of dataset characteristics is required.  

### ASPA Coverage

ASPA coverage characterizes the availability of provider authorization information for ASes involved in the verification workload.  

For benchmarking purposes, ASPA coverage is defined relative to a specified reference AS population derived from the BGP AS_PATHs and verification procedures used in the test.  

Let A_W denote the set of distinct ASNs that may be used as the customer argument in provider authorization checks for the specified workload and verification directions.  

Let A_D denote the set of CASes for which ASPA information is available in the dataset.
ASPA coverage is defined as:

    C_ASPA = (|A_W intersect A_D| / |A_W|) * 100%

The reference population A_W MUST remain unchanged when comparing different ASPA coverage levels for the same workload.
ASPA records unrelated to the reference population MUST NOT contribute to the coverage numerator.
If the reference population is empty, ASPA coverage is undefined and MUST be reported as not applicable.

The tester SHOULD evaluate multiple ASPA coverage levels to characterize how verification performance changes with the availability of authorization information.
Illustrative coverage levels include 0%, 25%, 50%, 75%, and 100%. Other levels MAY be used depending on the test objective.
When constructing datasets with different coverage levels, the selection of covered CASes MUST be reproducible.
Changing ASPA coverage may change the expected verification results of the test routes. Therefore, the distribution of Valid, Invalid, and Unknown results MUST be determined and reported for each coverage level.
An ASPA entry containing only AS 0 counts as an available ASPA entry for coverage purposes. It MUST NOT be treated as equivalent to the absence of ASPA information.

### AS_PATH Length and Structure
AS_PATH length is a primary characteristic of ASPA verification workloads.  

The ASPA verification procedures operate on the COMPRESSED_AS_PATH, obtained by removing consecutive duplicate ASNs from the AS_PATH.  

Therefore, the benchmarking workload MUST distinguish between:  
- Raw AS_PATH Length: The number of ASNs in the original AS_PATH.  
- Compressed AS_PATH Length: The number of ASNs remaining after consecutive duplicate ASNs have been removed.  

Unless otherwise specified, AS_PATH length in the benchmarking procedures refers to compressed AS_PATH length.
The tester SHOULD evaluate multiple compressed AS_PATH lengths to characterize their impact on verification performance.
For example, compressed path lengths of 2, 4, 8, 16, and 32 ASNs MAY be used as controlled test values. These values are illustrative rather than mandatory.
When evaluating AS_PATH length sensitivity, the tester SHOULD maintain consistent ASPA dataset characteristics, verification direction, and verification-result distribution where feasible.

In addition to path length, the structure of provider authorization relationships along the AS_PATH may influence verification processing.
The ASPA verification procedures distinguish between an up-ramp and a down-ramp based on the authorization relationships between consecutive ASes.
Therefore, test workloads SHOULD include different authorization relationship patterns consistent with the applicable verification procedures.

For tests intended to isolate the effect of AS_PATH prepending, the tester SHOULD construct paths with different raw AS_PATH lengths but identical compressed AS_PATHs.
This allows the processing overhead associated with AS_PATH representation and compression to be distinguished from the effects of compressed path length.
The raw and compressed AS_PATH length distributions MUST be reported for each applicable test.

### Provider Authorization Patterns

ASPA verification evaluates the authorization relationships between ASes appearing in an AS_PATH.
The provider authorization function defined in {{I-D.ietf-sidrops-aspa-verification}} produces three possible results:
- Provider+. 
- Not Provider+. 
- No Attestation. 

The distribution and position of these results along an AS_PATH may influence the verification outcome and the processing required by an implementation.
Therefore, the benchmarking methodology SHOULD consider the following authorization patterns:  
- Fully Authorized Paths: Paths for which the relevant provider authorization checks return Provider+ and the complete verification procedure produces Valid.  
- Paths with Missing Attestations: Paths containing authorization checks that return No Attestation, potentially resulting in an Unknown verification outcome.  
- Paths with Unauthorized Relationships: Paths containing authorization checks that return Not Provider+, potentially resulting in an Invalid verification outcome.  

The terms above describe workload characteristics and do not replace the verification results defined in {{I-D.ietf-sidrops-aspa-verification}}.
The tester SHOULD construct workloads in which the positions of missing attestations and unauthorized relationships are controlled.
For example, a test MAY place the first relevant No Attestation or Not Provider+ result at different positions along an AS_PATH while keeping the compressed path length unchanged.

The position MUST be described relative to the path orientation and authorization checks defined by the applicable upstream or downstream verification procedure.
Such tests may help characterize the sensitivity of an implementation to authorization relationship patterns and early termination opportunities.  

However, the final verification result MUST always be determined using the complete ASPA verification procedure. 
A single No Attestation or Not Provider+ result MUST NOT be assumed to produce a particular final verification result independently of the remaining path structure.

### Verification Direction and Outcome Distribution

ASPA verification applies different procedures depending on the relationship between the receiving router and its BGP neighbor.  
The benchmarking methodology distinguishes between:  
- Upstream Verification: Applied to routes received from Customers or Peers, and to the applicable Route Server and RS-client cases.  
- Downstream Verification: Applied to routes received from Providers.  

The tester SHOULD evaluate upstream and downstream verification separately before introducing workloads involving multiple verification directions.  
The configured neighbor relationship and the applied verification procedure MUST be reported.  
For each test workload, the expected ASPA verification results MUST be determined according to {{I-D.ietf-sidrops-aspa-verification}}.
The verification outcome distribution describes the proportion of routes expected to produce each result:

D=(p_{\mathrm{Valid}},p_{\mathrm{Invalid}},p_{\mathrm{Unknown}})

where: p_{\mathrm{Valid}}+p_{\mathrm{Invalid}}+p_{\mathrm{Unknown}}=1

The tester SHOULD include workloads dominated by individual verification results, as well as mixed workloads containing specified proportions of Valid, Invalid, and Unknown routes.  
The verification outcome distribution MUST be reported for each test.  
Where relevant, the outcome distribution across distinct AS_PATHs SHOULD also be reported separately from the route-weighted distribution.  
Not all combinations of ASPA coverage, AS_PATH structure, verification direction, and desired outcome distribution are necessarily achievable.  
The tester MUST ensure that the generated workload is consistent with the applicable ASPA verification procedures and MUST document any constraints that prevent the construction of a requested workload.  

### BGP Route and Unique AS_PATH Populations

The number of BGP routes and the number of distinct AS_PATHs are separate workload parameters.
Multiple routes may share the same AS_PATH, while other workloads may contain a large number of distinct AS_PATHs.
The benchmarking methodology MUST distinguish between:
- Total BGP route count.  
- Unique AS_PATH count.  
- Number of routes sharing each AS_PATH.  
- Number of distinct ASNs represented in the workload.  

These characteristics may influence the processing and memory requirements of ASPA verification implementations.
For example, an implementation may reuse previously computed verification results or maintain shared representations for routes with identical AS_PATHs.
The benchmarking methodology does not prescribe such optimizations. However, workloads SHOULD be constructed to allow their performance effects to be observed.
When evaluating the effect of unique AS_PATH count, the total number of BGP routes SHOULD remain fixed where feasible.
Conversely, when evaluating the effect of BGP route count, the number and characteristics of distinct AS_PATHs SHOULD remain fixed where feasible.
The tester MUST specify whether AS_PATH uniqueness is determined using raw or compressed AS_PATHs.

### ASPA Update Characteristics
ASPA updates may modify provider authorization information without changing the BGP routes currently stored by the DUT.
The benchmarking methodology distinguishes three ASPA update operations:  
- Addition: Introduction of an ASPA record for a previously uncovered CAS.  
- Replacement: Modification of the complete provider set associated with an existing CAS.  
- Withdrawal: Removal of an existing ASPA record for a CAS.  
The update workload MUST conform to the applicable RTR protocol semantics.  

For each update transaction, the following characteristics SHOULD be controlled and reported:  
- Number of changed CAS entries.  
- Number of provider authorizations added or removed.  
- Number of ASPA records announced or withdrawn.  
- Update transaction size.  
- Update transaction frequency.  
- Distribution of updates across CAS entries.  

When evaluating ASPA churn, the tester MUST specify whether the update rate refers to changed CAS records, individual provider authorization changes, or complete RTR update transactions.
These measures MUST NOT be treated as interchangeable.
The tester SHOULD evaluate both individual ASPA updates and batches containing multiple changes.
For revalidation-related benchmarks, the tester SHOULD also distinguish:
- The number of existing routes containing ASes relevant to the modified ASPA information.  
- The number of routes whose expected verification results change.  
- The number of distinct AS_PATHs whose expected verification results change.  

These quantities may differ because an ASPA update can affect multiple AS_PATHs, and multiple routes may share the same AS_PATH.
The affected route population MUST be determined using the expected verification results before and after the ASPA update.

### BGP Update Characteristics

The BGP update workload determines the rate and pattern at which routes requiring ASPA verification are presented to the DUT.
The benchmarking methodology SHOULD control the following parameters:  
- Number of BGP announcements and withdrawals.  
- BGP UPDATE message rate.  
- Route announcement and withdrawal rate.  
- Number of distinct AS_PATHs introduced per unit time.  
- Distribution of verification results among updated routes.  
- Burst size and inter-arrival intervals.  

The number of BGP UPDATE messages and the number of routes represented by those messages MUST be distinguished when calculating processing rates.
The tester SHOULD evaluate both constant-rate and bursty BGP update workloads.
For concurrent ASPA and BGP update tests, the timing relationship between the two update sources MUST be specified.
The RTR and BGP traffic generators SHOULD provide synchronized or otherwise correlated timestamps sufficient to reproduce the intended update sequence.

## ASPA Workload Characteristics

The input parameters defined in Section 5.2 can be combined to construct workloads that exercise different aspects of ASPA verification.
To support systematic and reproducible benchmarking, this document identifies four workload categories.

### Static Verification Workloads

Static verification workloads evaluate AS_PATH verification performance while the ASPA dataset remains unchanged.
The tester first installs a predefined ASPA dataset and allows the DUT to reach steady state. BGP routes are then introduced with controlled AS_PATH characteristics and expected verification results.
These workloads are intended to characterize the performance effects of:  
- Compressed AS_PATH length.  
- ASPA coverage.  
- Provider-set cardinality.  
- Provider authorization patterns.  
- Upstream and downstream verification procedures.  
- Verification outcome distributions.  
- Unique AS_PATH count and route sharing.  

When evaluating an individual parameter, the tester SHOULD maintain the remaining workload characteristics as consistently as possible.
If changes to one parameter necessarily affect other parameters, such dependencies MUST be documented.

### ASPA Update and Revalidation Workloads

ASPA update and revalidation workloads evaluate DUT behavior when ASPA information changes while the BGP route population remains stable.
The tester MUST establish a predefined ASPA dataset and BGP route population before triggering the ASPA update.
The update workload SHOULD include controlled changes that produce known verification-state transitions.
Examples include:  
- Valid to Invalid.  
- Invalid to Valid.  
- Unknown to Valid.  
- Unknown to Invalid.  
- Valid to Unknown.  
- Invalid to Unknown.  

These transitions MUST be constructed consistently with the applicable ASPA verification procedures.
The tester SHOULD independently control the number of changed ASPA records, the number of affected BGP routes, and the number of distinct affected AS_PATHs where feasible.
Workloads SHOULD include both concentrated updates, in which a small number of CAS changes affect many routes, and distributed updates, in which changes to multiple CAS entries affect different route populations.
These workloads are intended to characterize ASPA update processing and route revalidation performance independently of new BGP announcements.

### Concurrent Update Workloads

Concurrent update workloads evaluate ASPA verification performance when ASPA information and BGP routing information change during overlapping intervals.
The tester SHOULD specify independent ASPA and BGP update sequences.
The following characteristics MUST be documented:  
- ASPA update rate.  
- BGP update rate.  
- Duration of the concurrent workload.  
- Timing relationship between ASPA and BGP updates.  
- Number of routes affected by each workload.  
- Expected verification and routing states after the workload completes.  

Concurrent workloads SHOULD include both sustained update rates and controlled bursts.
When comparing concurrent workloads with isolated workloads, the tester SHOULD use equivalent ASPA and BGP update sequences to characterize the additional processing effects of simultaneous activity.

### Recovery Workloads

Recovery workloads evaluate ASPA verification behavior following RTR session interruptions, cache resets, or other controlled recovery events.
The tester MUST establish a known baseline ASPA dataset and BGP route population before triggering the recovery event.
Recovery workloads SHOULD distinguish between cases in which the ASPA dataset remains unchanged and cases in which the dataset changes during the interruption.
The tester SHOULD document whether previously received ASPA information remains available during recovery and identify the applicable RTR timing parameters.
Recovery measurements SHOULD include the time required to restore the expected ASPA dataset, verification results, and BGP routing state.

## Metrics and Measurements

This section defines the performance metrics used to characterize ASPA verification implementations.
For each metric, the benchmark MUST identify the measurement start event, completion event, and observation method.
Measurements obtained using internal DUT instrumentation MUST be distinguished from those obtained through externally observable protocol or routing-state events.

### ASPA Synchronization Time

ASPA synchronization time characterizes the time required for the DUT to receive and process an initial ASPA dataset.
The following measurements are distinguished:
RTR Data Transfer Time: The interval between the arrival of the first PDU of the measured RTR response at the DUT and the completion of the corresponding End of Data PDU.
ASPA Dataset Application Time: The instant at which the complete received ASPA dataset becomes available for ASPA verification.
End-to-End ASPA Synchronization Time: The interval between the beginning of the measured RTR response at the DUT and the point at which the complete ASPA dataset is available for verification.
The completion of an RTR transaction does not necessarily indicate the completion of internal ASPA processing.
Therefore, RTR data transfer time MUST NOT be reported as equivalent to end-to-end ASPA synchronization time.
When ASPA dataset application cannot be directly observed, the tester MUST report only the measurements supported by the available observation points.
The report MUST identify whether the measurement includes RTR session establishment, cache response generation, or other protocol operations.

### ASPA Update Processing Latency

ASPA update processing latency characterizes the time required for changes in ASPA information to become effective at the DUT.
For a controlled update transaction, the measurement begins when the corresponding RTR update response starts arriving at the DUT.
The measurement ends when the updated ASPA information becomes available for verification, as determined by the selected observation method.
The tester SHOULD distinguish:
- RTR update transfer time.
- End-to-end ASPA update processing latency.
- Post-transfer processing latency, when independently observable.
The measurement MUST specify the number and types of ASPA changes included in the transaction.
If the DUT applies ASPA information before the End of Data PDU is received, the tester MUST account for that behavior and MUST NOT assume that all processing begins after transfer completion.
Measurements involving subsequent route revalidation MUST be reported separately from ASPA data processing latency.

### AS_PATH Verification Latency
AS_PATH verification latency characterizes the time required for the DUT to determine the ASPA verification result of a BGP route.
Where suitable internal instrumentation is available, verification latency MAY be measured as the interval between the beginning and completion of the ASPA verification procedure for the route.
Such measurements MUST identify the corresponding instrumentation points.
When direct verification latency cannot be observed, the tester MAY measure externally observable verification-related route-processing latency.
This measurement begins with the arrival of the relevant BGP UPDATE at the DUT and ends at a defined observable event indicating that the route has reached the expected verification-related state.
The externally observable interval may include BGP message parsing, routing policy processing, queueing, and other control-plane operations.
Therefore, it MUST NOT be reported as the execution time of the ASPA verification algorithm.
The tester SHOULD report latency distributions rather than only aggregate averages when sufficient samples are available.

### Verification Throughput
Verification throughput characterizes the rate at which the DUT processes BGP routes requiring ASPA verification under a specified workload.
Where internal verification operation counts are available, verification throughput MAY be reported in verification operations per second.
Otherwise, the tester SHOULD use externally observable route-processing throughput.
For externally observable throughput measurements, the workload MUST be constructed so that the relevant routes reach identifiable expected states.
The tester SHOULD determine the maximum sustainable processing rate for which the DUT continues to process the workload without persistent backlog growth or unsuccessful route processing.
The measurement MUST distinguish:
- BGP UPDATE messages per second.
- Route announcements processed per second.
- Distinct AS_PATHs processed per second.
- Internal ASPA verification operations per second, when measurable.
These quantities MUST NOT be used interchangeably.
The benchmark MUST identify which quantity is being measured and the conditions used to determine successful processing.

### Route Revalidation Time
Route revalidation time characterizes the time required for previously received BGP routes to reach their expected verification states following an ASPA update.
The benchmark MUST define the affected route population before initiating the update.
Two measurements are distinguished:
End-to-End Route Revalidation Time: The interval between the beginning of the corresponding RTR update response at the DUT and completion of the expected verification-state changes for all affected routes.
Post-Application Route Revalidation Time: The interval between the point at which the updated ASPA information becomes available for verification and completion of the expected verification-state changes.
The second measurement MAY be reported only when both observation points are available.
The tester MUST identify how completion of route revalidation is determined.
The report MUST include:
- Number of changed ASPA records.
- Number of affected routes.
- Number of distinct affected AS_PATHs.
- Expected verification-state transitions.
- Route revalidation time.
When a test includes routes whose verification results remain unchanged, their number SHOULD be reported separately from the population whose results change.

### BGP Convergence Time
BGP convergence time characterizes the interval required for the DUT to reach the expected stable routing state following a specified event.
Two categories are considered:
ASPA-triggered BGP Convergence: BGP routing-state changes caused by an update to ASPA information.
BGP-triggered Convergence with ASPA Verification: BGP convergence following a controlled BGP routing event while ASPA verification is enabled.
For ASPA-triggered convergence, the measurement begins at the specified RTR update event and ends when the expected BGP routing state has been reached.
For BGP-triggered convergence, the measurement begins at the specified BGP event and ends when the expected stable routing state has been reached.
The tester MUST document the routing policies applied to ASPA verification results.
A change in ASPA verification result does not necessarily cause a BGP routing-state change. Therefore, ASPA-triggered BGP convergence time MUST be measured only for test conditions in which the configured routing policy produces a defined and observable routing-state transition.
When comparing ASPA-enabled and ASPA-disabled configurations, the benchmark SHOULD use equivalent BGP workloads and expected routing outcomes.
Any differences in routing policy or expected convergence behavior MUST be documented.

### Resource Utilization
Resource utilization characterizes the CPU and memory resources consumed by ASPA verification under specified workload conditions.
At minimum, the following metrics SHOULD be collected:
- Total CPU utilization.
- Total memory consumption.
- Peak CPU utilization during the test.
- Peak memory consumption during the test.
Where available, the tester SHOULD also collect:
- CPU utilization of routing processes.
- Memory consumption of routing processes.
- Memory used to store ASPA information.
- ASPA-related processing queue or backlog information.
Resource-utilization measurements SHOULD be collected during both steady-state and transient workload conditions.
The measurement interval, data source, and scope of the reported resource utilization MUST be documented.
For tests comparing configurations with ASPA verification enabled and disabled, resource utilization SHOULD be measured under equivalent BGP workload conditions.

### Scalability and Recovery Measurements
ASPA scalability is characterized by the maximum workload size that the DUT can process while satisfying predefined successful-operation criteria.
Relevant scalability dimensions include:
- ASPA record count.
- Provider authorization count.
- Provider-set cardinality.
- BGP route population.
- Unique AS_PATH count.
- Compressed AS_PATH length.
- ASPA update rate.
- Number of routes affected by ASPA updates.
The tester MUST specify the scalability dimension being evaluated and the criteria used to determine successful operation.
These criteria MAY include successful completion of ASPA synchronization, correct verification results, stable BGP sessions, bounded processing latency, and the absence of persistent processing backlogs.
A reported scalability limit MUST identify whether it represents an observed DUT limitation, a predefined performance threshold, or the maximum workload supported by the test equipment.
Recovery-related measurements SHOULD include RTR reconnection time, ASPA resynchronization time, and the time required to restore the expected verification and routing states.
The recovery scenario, initial conditions, and applicable RTR timing parameters MUST be documented.

# Benchmark Tests

This section defines the benchmark tests for evaluating the performance and operational behavior of ASPA-based AS_PATH verification implementations.

The tests cover ASPA data processing, AS_PATH verification, route revalidation, BGP convergence, scalability, resource utilization, and RTR recovery behavior.

Unless otherwise specified, each test uses the laboratory environment described in Section 4 and the workload parameters and measurement definitions specified in Section 5.

Each benchmark specifies its objective, test conditions, procedure, measurements, and reporting requirements.

The DUT MUST reach the predefined initial state before each test. The expected ASPA verification results MUST be determined independently of the DUT using the procedures defined in {{I-D.ietf-sidrops-aspa-verification}}. Test runs that do not reach the expected final state MUST be identified and MUST NOT be reported as successful benchmark results.

For timing measurements, the following conventions apply:

* RTR input events are observed at the DUT-facing interface. The beginning of an RTR response is identified by the arrival of its Cache Response PDU, and transfer completion is identified by the arrival of the corresponding End of Data PDU.

* BGP input events are observed at the DUT-facing interface. The arrival of a BGP UPDATE is identified by the completion of the corresponding message at the observation point.

* Internal processing completion is determined using DUT instrumentation or management interfaces when available.

* Externally observable completion is determined using predefined verification-state observations or routing-state changes. The observation method and its limitations MUST be documented.

* Measurements obtained from internal instrumentation MUST be distinguished from externally observable measurements. In particular, externally observable route-processing latency MUST NOT be reported as the execution time of the ASPA verification algorithm.

Where direct verification-state information is unavailable, the tester MAY use a controlled routing policy and observable BGP routing-state transitions as an indirect measurement. Such measurements MUST be identified as verification-related routing-state response times rather than direct ASPA verification latencies.

If a test requires an internal completion event that cannot be observed, the corresponding internal metric MUST be reported as unavailable. Other measurable results from the same test MAY still be reported.

Each test SHOULD be repeated under identical conditions, as specified in Section 5.1. The report MUST include the number of repetitions, measurement variability, and any unsuccessful runs.

## ASPA Data Processing Latency

**Objective:**

Measure the time required for the DUT to receive and process ASPA information delivered through the RPKI-to-Router (RTR) protocol.

Two scenarios are considered: initial ASPA dataset synchronization and incremental ASPA updates.

### Initial ASPA Synchronization

This test characterizes ASPA processing performance when the DUT loads a complete ASPA dataset from an RTR cache.

**Test Conditions:**

The DUT MUST be configured with ASPA verification enabled and connected to the RTR Cache/Emulator described in Section 4.3.

The tester MUST prepare an ASPA dataset with specified CAS and provider authorization counts.

The test MUST begin from a known RTR synchronization state. The tester MUST ensure that previously installed ASPA information does not unintentionally affect the measurements.

The test MAY be conducted with or without an existing BGP route population. When BGP routes are present, their number and AS_PATH characteristics MUST be documented.

**Procedure:**

1. Configure the DUT and RTR Cache/Emulator according to the selected test parameters.

2. Prepare an ASPA dataset with the specified CAS count, provider authorization count, and provider-set cardinality distribution.

3. Establish the required BGP sessions and baseline route population, if applicable.

4. Initiate a full RTR synchronization using the applicable Reset Query procedure.

5. Record the arrival time of the Cache Response PDU at the DUT-facing interface.

6. Record the arrival time of the corresponding End of Data PDU.

7. Determine when the received ASPA dataset becomes available for verification, if a suitable observation mechanism is available.

8. Record CPU and memory utilization during synchronization.

9. Verify that the expected ASPA information and verification states are available after synchronization.

10. Restore the initial conditions and repeat the test.

The tester SHOULD repeat the procedure using different ASPA dataset sizes.

The number of CAS entries and the total number of provider authorizations SHOULD be varied independently where feasible.

**Measurements:**

* RTR data transfer time.
* End-to-end ASPA synchronization time, when observable.
* ASPA application completion time, when observable.
* Peak CPU utilization.
* Peak memory consumption.
* Number of successfully installed ASPA records, when observable.
* Synchronization failures.

The completion of the End of Data PDU indicates completion of the corresponding RTR data transfer but does not necessarily indicate that all ASPA information has become effective for route verification.

Therefore, RTR data transfer time MUST NOT be reported as ASPA processing completion time.

When direct ASPA application completion cannot be observed, the tester MAY measure the time required for predefined routing-state changes resulting from the newly synchronized ASPA information. Such measurements MUST be reported separately as externally observable routing-state response times.

**Reporting:**

The report MUST include the ASPA dataset characteristics, RTR protocol version, measurement observation points, and synchronization results.

When other RTR payload types are present, their quantities MUST be reported. Non-ASPA payload populations SHOULD remain unchanged across test runs intended to isolate ASPA processing performance.

### Incremental ASPA Update Processing

This test measures DUT processing performance when an existing ASPA dataset is modified.

**Test Conditions:**

The DUT MUST initially contain a predefined ASPA dataset and have reached steady state.

The RTR Cache/Emulator MUST support controlled ASPA additions, replacements, and withdrawals.

Each update transaction MUST conform to the applicable RTR protocol semantics.

In particular, a replacement announcement for an existing CAS MUST contain the complete new provider set.

**Procedure:**

1. Load the baseline ASPA dataset and establish steady state.

2. Prepare a controlled ASPA update transaction containing a specified number of changes.

3. Select one of the following update operations:

   * Addition of new CAS entries.
   * Replacement of existing CAS provider sets.
   * Withdrawal of existing CAS entries.

4. Make the updated dataset available through the RTR Cache/Emulator.

5. Initiate the corresponding RTR update exchange using the applicable protocol procedures.

6. Record the arrival time of the Cache Response PDU at the DUT-facing interface.

7. Record the arrival time of the corresponding End of Data PDU.

8. Determine when the updated ASPA information becomes effective, if directly observable.

9. Record processing latency and resource utilization.

10. Verify that the resulting ASPA dataset matches the expected state.

11. Restore the baseline dataset and repeat the test.

The tester SHOULD evaluate additions, replacements, and withdrawals separately before using mixed update transactions.

The tester SHOULD vary the number of changed CAS entries and provider authorizations across test runs.

**Measurements:**

* RTR update transfer time.
* ASPA update processing latency, when observable.
* Externally observable routing-state response time, when applicable.
* Number of changed CAS entries.
* Number of provider authorizations added or removed.
* CPU and memory utilization.
* Update processing failures.

**Reporting:**

Results SHOULD characterize update processing performance as a function of the number of changed CAS entries and provider authorizations.

The report MUST distinguish RTR transfer completion, ASPA data application, and subsequent route revalidation.

When an ASPA update triggers route revalidation, the corresponding revalidation performance SHOULD be measured using Section 6.3.

## AS_PATH Verification Performance

**Objective:**

Characterize ASPA verification performance under controlled BGP workloads with different AS_PATH structures and ASPA dataset characteristics.

These tests evaluate the effects of AS_PATH length, ASPA coverage, provider-set cardinality, provider authorization patterns, verification direction, and verification outcome distribution.

Unless otherwise specified, the ASPA dataset MUST remain stable while BGP routes are introduced.

### Baseline AS_PATH Verification

This test establishes baseline measurements for BGP route processing with ASPA verification enabled.

**Test Conditions:**

The DUT MUST be configured with ASPA verification enabled.

The tester MUST prepare an ASPA dataset and a BGP workload with known verification results.

The workload MUST specify:

* Number of BGP routes.
* Number of unique AS_PATHs.
* Compressed AS_PATH length distribution.
* ASPA coverage.
* Verification direction.
* Verification outcome distribution.
* BGP route announcement rate.

**Procedure:**

1. Load the predefined ASPA dataset into the DUT.

2. Establish the BGP sessions required for the selected verification direction.

3. Confirm completion of ASPA synchronization and establish steady state.

4. Prepare BGP routes with predefined AS_PATHs and expected verification results.

5. Start the Measurement System.

6. Announce the test routes at the specified rate.

7. Record the relevant BGP input events and observable verification or routing-state changes.

8. Measure verification-related processing latency and route-processing throughput using the observation methods defined in Section 5.4.

9. Verify that the DUT produces the expected verification results.

10. Withdraw the test routes, restore the initial state, and repeat the test.

The tester SHOULD conduct a corresponding measurement with ASPA verification disabled.

The ASPA-enabled and ASPA-disabled configurations SHOULD use equivalent BGP workloads and routing policies that preserve the same expected route-selection outcomes when the objective is to isolate ASPA processing overhead.

**Measurements:**

* Internal ASPA verification latency, when directly measurable.
* Externally observable verification-related route-processing latency.
* Route-processing throughput.
* CPU utilization.
* Memory consumption.
* Processing overhead relative to the ASPA-disabled baseline.

**Reporting:**

The report MUST identify the measurement method and distinguish internal verification latency from externally observable processing latency.

Results SHOULD include latency distributions and throughput measurements under the specified workload.

### AS_PATH Length Sensitivity

This test evaluates how verification performance varies with compressed AS_PATH length.

**Procedure:**

1. Prepare multiple BGP workloads with different compressed AS_PATH lengths.

2. Construct corresponding ASPA datasets that produce known verification results.

3. Maintain the number of advertised routes and other relevant workload characteristics as consistently as possible.

4. Load the ASPA dataset corresponding to the first workload.

5. Establish steady state.

6. Announce the test routes at a fixed rate.

7. Measure verification-related processing latency, throughput, and resource utilization.

8. Restore the initial state.

9. Repeat the procedure for each selected AS_PATH length.

The tester SHOULD evaluate upstream and downstream verification separately.

Where feasible, the verification outcome distribution SHOULD remain consistent across the tested path lengths.

The tester MAY also construct workloads with different raw AS_PATH lengths but identical compressed AS_PATHs to characterize the additional processing associated with AS_PATH prepending.

**Measurements:**

* Verification-related processing latency.
* Route-processing throughput.
* CPU and memory utilization.

**Reporting:**

Results SHOULD be presented against compressed AS_PATH length.

Raw AS_PATH length MUST also be reported when AS_PATH prepending is present.

Differences in ASPA dataset characteristics or verification outcome distributions across test conditions MUST be documented.

### ASPA Coverage Sensitivity

This test evaluates the effect of ASPA coverage on verification performance.

**Procedure:**

1. Prepare a fixed population of BGP routes and AS_PATHs.

2. Determine the reference AS population used to calculate ASPA coverage, as defined in Section 5.2.2.

3. Generate ASPA datasets with different coverage levels.

4. Determine the expected verification results for each dataset.

5. Load the first ASPA dataset and establish steady state.

6. Announce the predefined BGP workload.

7. Measure verification-related latency, throughput, and resource utilization.

8. Restore the initial state.

9. Repeat the procedure for each coverage level.

The BGP route population, AS_PATH characteristics, and verification direction SHOULD remain unchanged across coverage levels.

Because changes in ASPA coverage may affect the verification outcome distribution, the expected and observed outcome distributions MUST be reported for each dataset.

**Measurements:**

* Verification-related processing latency.
* Route-processing throughput.
* CPU and memory utilization.
* Verification outcome distribution.

**Reporting:**

Results SHOULD characterize verification performance as a function of ASPA coverage.

The report MUST identify the reference AS population and the method used to select covered CASes.

### Provider-Set Cardinality Sensitivity

This test evaluates the influence of provider-set cardinality on ASPA verification performance.

**Procedure:**

1. Prepare ASPA datasets containing the same number of CAS entries but different provider-set cardinality distributions.

2. Construct BGP workloads that exercise the corresponding provider authorization relationships.

3. Determine the expected verification results for each workload.

4. Load the first ASPA dataset and establish steady state.

5. Announce the test routes at a fixed rate.

6. Measure verification-related latency, throughput, and resource utilization.

7. Restore the baseline state.

8. Repeat the procedure for the remaining provider-set cardinality distributions.

The tester SHOULD maintain equivalent BGP route counts, compressed AS_PATH length distributions, and verification directions across test runs.

When increasing provider-set cardinality also increases the total provider authorization count, the resulting dataset-size difference MUST be documented.

**Measurements:**

* Verification-related processing latency.
* Route-processing throughput.
* CPU and memory utilization.

**Reporting:**

Results SHOULD characterize verification performance as a function of provider-set cardinality.

The report MUST include the provider-set cardinality distribution and total provider authorization count for each dataset.

### Provider Authorization Pattern Sensitivity

This test evaluates whether the distribution and position of provider authorization results along an AS_PATH influence verification performance.

The purpose is to characterize implementation performance without requiring a particular internal verification algorithm or optimization.

**Procedure:**

1. Select a fixed compressed AS_PATH length and verification direction.

2. Construct multiple ASPA datasets and corresponding BGP workloads with different provider authorization patterns.

3. Control the positions of Provider+, Not Provider+, and No Attestation results along the AS_PATH.

4. Determine the expected final verification result for each workload using {{I-D.ietf-sidrops-aspa-verification}}.

5. Maintain equivalent route counts and compressed AS_PATH lengths across test conditions.

6. Load the selected ASPA dataset and establish steady state.

7. Announce the corresponding BGP workload.

8. Measure verification-related processing latency, throughput, and resource utilization.

9. Restore the baseline state.

10. Repeat the procedure for each authorization pattern.

The tester SHOULD include patterns in which the first relevant Not Provider+ or No Attestation result occurs at different positions along the path.

Where feasible, the workloads SHOULD maintain the same final verification outcome while varying the authorization pattern.

The positions of authorization results MUST be defined consistently with the AS_PATH orientation and the applicable upstream or downstream verification procedure.

The tester MUST NOT assume that an individual provider authorization result alone determines the final AS_PATH verification outcome.

**Measurements:**

* Verification-related processing latency.
* Route-processing throughput.
* CPU and memory utilization.

**Reporting:**

The report MUST describe the authorization patterns and expected verification results.

Results SHOULD characterize performance differences across the tested authorization patterns.

### Upstream and Downstream Verification

This test evaluates verification performance under the upstream and downstream procedures defined in {{I-D.ietf-sidrops-aspa-verification}}.

**Procedure:**

1. Configure an eBGP session with a neighbor relationship that invokes upstream verification.

2. Prepare an ASPA dataset and BGP workload with known verification results.

3. Load the dataset and establish steady state.

4. Announce the BGP workload.

5. Measure verification-related latency, throughput, and resource utilization.

6. Restore the baseline state.

7. Configure an eBGP session with a neighbor relationship that invokes downstream verification.

8. Prepare and load the corresponding ASPA dataset and BGP workload.

9. Repeat the measurements.

The tester SHOULD maintain equivalent route counts, compressed AS_PATH length distributions, and BGP update rates across the two scenarios.

Because upstream and downstream verification use different procedures, their provider authorization patterns and expected outcomes MUST be constructed separately.

**Measurements:**

* Verification-related processing latency.
* Route-processing throughput.
* CPU and memory utilization.

**Reporting:**

The report MUST identify the neighbor relationship, verification direction, AS_PATH characteristics, ASPA dataset characteristics, and verification outcome distribution.

Any workload differences between the two verification directions MUST be documented.

### Sustained Verification Throughput

This test determines the maximum sustainable BGP route-processing rate with ASPA verification enabled.

**Test Conditions:**

The DUT MUST contain a stable ASPA dataset.

The BGP Route Generator MUST be capable of producing a sufficiently large population of routes with known verification results.

Before the test, the tester MUST define the conditions for successful sustained processing.

**Procedure:**

1. Prepare a predefined ASPA dataset and BGP route population.

2. Load the ASPA dataset and establish steady state.

3. Configure the BGP Route Generator with an initial announcement rate.

4. Start announcing routes at the selected rate for a predefined measurement interval.

5. Record the offered route rate and the rate at which routes reach their expected observable states.

6. Measure processing latency and resource utilization.

7. Determine whether the DUT satisfies the predefined successful-operation criteria.

8. Restore the initial state and repeat at a higher announcement rate.

9. Continue until the maximum planned rate is reached or the DUT can no longer sustain the workload.

The successful-operation criteria MUST include a defined condition for determining that the offered workload has been processed.

Where processing backlog information is available, the tester SHOULD use it to determine whether the backlog remains bounded.

Where backlog information is unavailable, the tester SHOULD use externally observable processing completion and a predefined maximum completion delay.

The tester MUST ensure that the BGP generator does not become the limiting factor.

**Measurements:**

* Offered BGP route announcement rate.
* Successfully processed route rate.
* Verification-related processing latency.
* CPU and memory utilization.
* Processing backlog, when observable.
* Processing failures.

**Reporting:**

The maximum sustainable processing rate MUST be reported together with the workload characteristics and successful-operation criteria.

The report MUST distinguish offered throughput from successfully processed throughput.

The result MUST be identified as route-processing throughput unless internal ASPA verification operations are measured directly.

## ASPA-triggered Route Revalidation

**Objective:**

Measure the time required for the DUT to re-evaluate previously received BGP routes following changes in ASPA information.

This benchmark focuses on ASPA-triggered route revalidation without introducing additional BGP announcements or withdrawals.

The number of changed ASPA records, the number of affected routes, and the number of distinct affected AS_PATHs are treated as separate workload characteristics.

Where direct verification-state information is available, the tester SHOULD use it to determine route revalidation completion.

Otherwise, the tester MAY use controlled routing-state transitions as an indirect observation. Such measurements MUST be identified as externally observable routing-state response times.

### Verification-State Transition

This test characterizes revalidation performance for different verification-state transitions.

**Test Conditions:**

The DUT MUST contain a predefined ASPA dataset and BGP route population.

The tester MUST prepare an ASPA update that changes the expected verification results of a known set of existing routes.

The existing BGP routes MUST remain unchanged during the test.

**Procedure:**

1. Load the baseline ASPA dataset.

2. Advertise the predefined BGP route population.

3. Confirm the expected initial verification states.

4. Establish steady state.

5. Prepare an ASPA update that produces a specified verification-state transition.

6. Trigger the update through the RTR Cache/Emulator.

7. Record the corresponding RTR input events.

8. Determine when the updated ASPA information becomes effective, if observable.

9. Monitor the affected routes using the selected verification-state or routing-state observation mechanism.

10. Determine when the expected post-update states have been reached.

11. Record the corresponding response time and resource utilization.

12. Restore the baseline dataset and repeat the test.

The tester SHOULD evaluate the following transitions where they can be constructed consistently with the applicable verification procedures:

* Valid to Invalid.
* Invalid to Valid.
* Unknown to Valid.
* Unknown to Invalid.
* Valid to Unknown.
* Invalid to Unknown.

Each transition type SHOULD be evaluated separately before mixed-transition workloads are introduced.

When the selected routing policy makes Invalid routes ineligible, the absence of a route from the Loc-RIB or from a monitoring session MUST NOT, by itself, be interpreted as proof that ASPA revalidation has completed.

If the verification-state transition cannot be directly observed, the tester MAY construct an associated routing-state transition that is externally observable. The resulting measurement MUST be labeled accordingly.

**Measurements:**

* End-to-end route revalidation time, when directly observable.
* Post-ASPA-application revalidation time, when directly observable.
* Externally observable routing-state response time, when applicable.
* Number of changed ASPA records.
* Number of affected routes.
* Number of distinct affected AS_PATHs.
* Verification-state transition type.
* CPU and memory utilization.
* Revalidation or routing-state completion failures.

**Reporting:**

Results SHOULD characterize revalidation performance for each verification-state transition type.

The report MUST identify the observation method and distinguish direct revalidation measurements from indirect routing-state measurements.

### Affected Route Population Sensitivity

This test evaluates the effect of the number of affected BGP routes on ASPA-triggered revalidation performance.

**Procedure:**

1. Prepare a baseline ASPA dataset and BGP route population with known verification results.

2. Construct an ASPA update that changes the expected verification states of a controlled subset of routes.

3. Select an initial affected route population size.

4. Load the baseline ASPA dataset and advertise the corresponding routes.

5. Establish steady state.

6. Trigger the ASPA update.

7. Measure the time required for the affected routes to reach their expected observable post-update states.

8. Record CPU and memory utilization.

9. Restore the initial state.

10. Repeat with increasing affected route population sizes.

The tester SHOULD keep the number of changed ASPA records constant while varying the affected route population.

The compressed AS_PATH length distribution, verification direction, and verification-state transition distribution SHOULD remain equivalent across test conditions where feasible.

**Measurements:**

* Number of affected routes.
* Number of distinct affected AS_PATHs.
* Route revalidation time, when directly observable.
* Externally observable routing-state response time, when applicable.
* CPU and memory utilization.

**Reporting:**

Results SHOULD be presented against the affected route population size.

The report MUST include the number of distinct affected AS_PATHs and the observation method used.

### Unique AS_PATH Sensitivity

This test evaluates how the number of distinct AS_PATHs influences ASPA-triggered revalidation performance.

**Procedure:**

1. Prepare multiple BGP workloads containing the same total number of affected routes but different numbers of distinct AS_PATHs.

2. Construct corresponding ASPA datasets and updates that produce known verification-state changes.

3. Maintain equivalent route counts and verification-state transition distributions across workloads.

4. Load the first ASPA dataset and advertise the corresponding routes.

5. Establish steady state.

6. Trigger the ASPA update.

7. Measure revalidation-related response time and resource utilization.

8. Restore the baseline state.

9. Repeat using the remaining workloads.

The tester SHOULD consider workloads in which:

* Many routes share a small number of distinct AS_PATHs.
* Routes are distributed across a moderate number of distinct AS_PATHs.
* Most routes have distinct AS_PATHs.

Where feasible, the compressed AS_PATH length distribution and authorization patterns SHOULD remain equivalent across the tested workloads.

**Measurements:**

* Number of affected routes.
* Number of distinct affected AS_PATHs.
* Route revalidation time, when directly observable.
* Externally observable routing-state response time, when applicable.
* CPU and memory utilization.

**Reporting:**

Results SHOULD characterize revalidation-related performance as a function of distinct AS_PATH count.

The report MUST include the distribution of routes across distinct AS_PATHs.

This test does not require the DUT to implement verification-result caching, shared path representations, or any other specific optimization.

## BGP Convergence with ASPA Verification Enabled

**Objective:**

Evaluate the impact of ASPA verification on BGP control-plane convergence under controlled routing conditions.

Two scenarios are considered:

* ASPA-triggered BGP convergence.
* BGP-triggered convergence with ASPA verification enabled and disabled.

The convergence measurements in this section concern control-plane routing behavior. Data-plane convergence MAY be measured separately using an applicable methodology.

### ASPA-triggered BGP Convergence

This test evaluates BGP convergence caused by changes in ASPA verification results.

**Test Conditions:**

The DUT MUST be configured with an ASPA-aware routing policy that produces a deterministic and observable routing-state change following the selected ASPA update.

The tester MUST prepare a stable BGP route population containing routes whose eligibility or selection changes as a result of the update.

The initial and expected final routing states MUST be determined before testing.

**Procedure:**

1. Configure ASPA verification and the selected routing policy.

2. Load the baseline ASPA dataset.

3. Establish the BGP sessions and advertise the predefined routes.

4. Confirm the expected initial route-selection state.

5. Establish steady state.

6. Prepare an ASPA update that changes the eligibility or selection of a known set of routes.

7. Trigger the ASPA update.

8. Record the relevant RTR input events.

9. Monitor the routing states using the selected observation mechanism.

10. Determine when the expected stable post-update routing state has been reached.

11. Record convergence time and resource utilization.

12. Restore the initial state and repeat the test.

The tester SHOULD evaluate different numbers of affected prefixes and routes.

The test MAY include:

* A previously selected route becoming ineligible.
* A previously ineligible route becoming eligible.
* A change in the selected route among multiple candidates.

The selected routing policy MUST remain unchanged across comparable test runs.

**Measurements:**

* End-to-end ASPA-triggered BGP convergence time.
* Post-ASPA-application convergence time, when observable.
* Number of affected routes and prefixes.
* CPU and memory utilization.
* Convergence failures.

**Reporting:**

The report MUST describe the routing policy, initial routing state, expected final state, and convergence observation points.

A verification-state change that does not produce a routing-state transition MUST NOT be reported as a BGP convergence event.

### BGP-triggered Convergence with and without ASPA Verification

This test compares BGP convergence performance with ASPA verification disabled and enabled.

**Procedure:**

1. Prepare a predefined BGP topology and route population.

2. Configure the DUT with ASPA verification disabled.

3. Establish the initial routing state.

4. Trigger a controlled BGP routing event, such as:

   * Announcement of a predefined route population.
   * Withdrawal of selected routes.
   * Replacement of selected AS_PATHs.
   * Controlled BGP peer failure.

5. Measure the time required to reach the expected stable routing state.

6. Record CPU and memory utilization.

7. Restore the initial state.

8. Enable ASPA verification and load the predefined ASPA dataset.

9. Confirm that the expected initial routing state is equivalent to that used in the ASPA-disabled test.

10. Repeat the same BGP event.

11. Measure convergence time and resource utilization.

12. Compare the results.

To isolate the processing overhead of ASPA verification, the expected route-selection outcomes SHOULD remain equivalent between the two configurations.

The measurement start and completion conditions MUST be equivalent.

**Measurements:**

* BGP convergence time without ASPA verification.
* BGP convergence time with ASPA verification.
* Absolute convergence-time difference.
* Relative convergence-time difference.
* CPU and memory utilization.

The relative convergence-time difference MAY be calculated as:

    Relative Change = ((T_ASPA - T_Baseline) / T_Baseline) * 100%

where T_ASPA and T_Baseline are measured under equivalent test conditions.

**Reporting:**

The report MUST identify the BGP event, routing configuration, workload characteristics, and convergence criteria.

Results SHOULD include absolute convergence times, relative differences, and measurement variability.

## ASPA Scalability Tests

**Objective:**

Characterize ASPA verification performance as ASPA dataset size and BGP workload complexity increase.

The benchmark considers:

* ASPA record count.
* Provider authorization count.
* Provider-set cardinality.
* BGP route population.
* Unique AS_PATH count.
* Compressed AS_PATH length.
* Number of routes affected by ASPA updates.

### ASPA Dataset Scalability

This test evaluates DUT performance as the ASPA dataset size increases.

**Procedure:**

1. Generate ASPA datasets with progressively increasing CAS counts.

2. Maintain a controlled provider-set cardinality distribution.

3. Prepare a fixed BGP workload, where applicable.

4. Load the first ASPA dataset and establish steady state.

5. Measure synchronization performance, verification-related processing performance, and resource utilization.

6. Restore the initial state.

7. Repeat using larger ASPA datasets.

8. Continue until the maximum planned dataset size is reached or the DUT fails to satisfy the predefined successful-operation criteria.

The tester SHOULD conduct a separate series of measurements in which the provider authorization count increases while CAS count remains fixed.

The tester SHOULD distinguish ASPA records referenced by the current BGP workload from records unrelated to the current AS_PATH population.

**Measurements:**

* ASPA synchronization time.
* Verification-related processing latency.
* CPU and memory utilization.
* Successfully installed ASPA records, when observable.
* Processing failures.

**Reporting:**

Results SHOULD be presented against ASPA record count and provider authorization count.

The report MUST distinguish DUT limitations from test equipment limitations.

### BGP Route and AS_PATH Scalability

This test evaluates the impact of BGP route population and distinct AS_PATH count on ASPA verification performance.

**Procedure:**

1. Prepare a fixed ASPA dataset.

2. Generate BGP workloads with increasing route counts.

3. Define the number and distribution of distinct AS_PATHs for each workload.

4. Load the ASPA dataset and establish steady state.

5. Announce the selected BGP workload.

6. Measure processing throughput, latency, and resource utilization.

7. Restore the initial state.

8. Repeat using larger workloads.

The tester SHOULD evaluate route count and distinct AS_PATH count as separate scalability dimensions.

When isolating route-count effects, the number and characteristics of distinct AS_PATHs SHOULD remain fixed.

When isolating unique-AS_PATH effects, the total route count SHOULD remain fixed.

**Measurements:**

* Route-processing throughput.
* Verification-related processing latency.
* CPU and memory utilization.
* Processing failures.

**Reporting:**

The report MUST include route count, unique AS_PATH count, and the distribution of routes across distinct AS_PATHs.

Results SHOULD characterize performance changes across the tested workload sizes.

### Scalability Limit Determination

This test identifies the highest workload level at which the DUT satisfies predefined successful-operation criteria.

**Procedure:**

1. Select a scalability parameter and define its test range.

2. Establish successful-operation criteria before testing.

3. Execute the corresponding benchmark at the initial parameter value.

4. Increase the selected parameter while maintaining other relevant workload characteristics.

5. Determine whether the DUT satisfies the successful-operation criteria.

6. Continue until an unsuccessful condition occurs or the maximum planned test value is reached.

7. Record the highest successfully tested value.

8. Repeat measurements near the observed limit where feasible.

Successful-operation criteria SHOULD include:

* Successful ASPA synchronization, when applicable.
* Expected ASPA verification results.
* Stable BGP sessions.
* Completion of the specified processing workload.
* Absence of persistent processing backlogs.
* Absence of resource exhaustion or processing failures.

Any latency or resource-utilization thresholds used to define successful operation MUST be specified before testing.

**Reporting:**

The report MUST identify the tested scalability parameter, successful-operation criteria, highest successful test value, and observed failure conditions.

A result MUST NOT be described as the maximum DUT capacity when the test is limited by the workload generator or the maximum planned test size.

## ASPA Churn and Stress Tests

**Objective:**

Evaluate ASPA verification performance under sustained ASPA updates, bursty ASPA updates, and concurrent ASPA and BGP activity.

These tests characterize processing stability, latency, revalidation behavior, and resource utilization under update-intensive conditions.

### Sustained ASPA Churn

This test evaluates the DUT under a sustained sequence of ASPA dataset changes.

**Test Conditions:**

The DUT MUST contain a predefined ASPA dataset and BGP route population.

The RTR Cache/Emulator MUST support reproducible ASPA update sequences containing additions, replacements, and withdrawals.

The update workload MUST conform to the applicable RTR protocol procedures and timing constraints.

**Procedure:**

1. Load the baseline ASPA dataset and BGP route population.

2. Confirm the expected initial verification states.

3. Establish steady state.

4. Prepare an ASPA update sequence with a specified rate and duration.

5. Start the update sequence using the applicable RTR protocol procedures.

6. Record the rate of changes generated by the ASPA data source and the rate of changes delivered to the DUT.

7. Monitor ASPA processing, route revalidation behavior, and resource utilization throughout the test.

8. Stop generating new updates after the predefined test duration.

9. Determine whether the DUT processes all delivered updates and reaches the expected final verification state.

10. Record the time required to reach the final stable state.

11. Restore the baseline conditions and repeat at different update rates.

The tester SHOULD evaluate:

* Repeated additions and withdrawals.
* Replacements of existing provider sets.
* Updates distributed across many CAS entries.
* Repeated updates affecting a small CAS population.

The ASPA dataset size SHOULD remain approximately stable when the objective is to isolate update-frequency effects.

The tester MUST distinguish changes generated by the ASPA data source from RTR update transactions actually delivered to the DUT.

Multiple source-side dataset changes may be represented by a smaller number of RTR update transactions. These quantities MUST NOT be treated as interchangeable.

**Measurements:**

* Generated ASPA change rate.
* Delivered ASPA update rate.
* ASPA update processing latency, when observable.
* Route revalidation time, when observable.
* CPU and memory utilization.
* Processing backlog, when observable.
* Time to reach the final stable state.
* Processing failures.

**Reporting:**

Results SHOULD characterize processing performance and resource utilization against the delivered ASPA update rate.

The report MUST describe the update sequence, update types, test duration, and actual RTR transactions delivered.

### Burst ASPA Updates

This test evaluates the DUT when a large number of ASPA changes are delivered within a short interval.

**Procedure:**

1. Establish the baseline ASPA dataset and BGP route population.

2. Confirm steady state.

3. Prepare a controlled ASPA update transaction containing a specified number of changed CAS entries.

4. Trigger the corresponding RTR update exchange.

5. Record the RTR response timing.

6. Measure ASPA update processing performance using the available observation points.

7. Measure subsequent route revalidation behavior, when applicable.

8. Record peak resource utilization.

9. Verify that the DUT reaches the expected final state.

10. Restore the baseline conditions and repeat with different burst sizes.

The tester SHOULD distinguish between:

* A single RTR response containing many ASPA changes.
* Multiple consecutive RTR update transactions.

These patterns MUST NOT be treated as equivalent.

**Measurements:**

* Number of changed CAS entries.
* Number of affected provider authorizations.
* RTR update transfer time.
* ASPA update processing latency, when observable.
* Route revalidation time, when observable.
* Peak CPU and memory utilization.
* Processing failures.

**Reporting:**

Results SHOULD be presented against update burst size.

The report MUST identify whether the burst consists of one RTR transaction or multiple transactions.

### Concurrent ASPA and BGP Updates

This test evaluates DUT performance when ASPA information and BGP routing information change during overlapping intervals.

**Test Conditions:**

The tester MUST prepare independent, reproducible ASPA and BGP update sequences.

The expected final ASPA dataset, verification results, and BGP routing states MUST be determined before testing.

**Procedure:**

1. Load the baseline ASPA dataset.

2. Establish the baseline BGP route population.

3. Confirm the expected initial verification and routing states.

4. Prepare the ASPA update sequence.

5. Prepare the BGP announcement and withdrawal sequence.

6. Establish steady state.

7. Start the two update sequences according to a predefined timing relationship.

8. Record ASPA and BGP input events and corresponding observable processing behavior.

9. Measure processing latency, throughput, and resource utilization during the test.

10. Stop both update sequences after their specified durations.

11. Determine whether the DUT reaches the expected final ASPA and BGP states.

12. Restore the baseline conditions and repeat using different combinations of update rates.

The tester SHOULD evaluate:

* ASPA updates beginning before BGP updates.
* BGP updates beginning before ASPA updates.
* ASPA and BGP updates beginning within the same controlled interval.

The tester SHOULD also conduct corresponding ASPA-only and BGP-only tests using equivalent update sequences.

These isolated measurements provide a baseline for evaluating the additional effects of concurrent workloads.

**Measurements:**

* ASPA update rate.
* BGP route announcement and withdrawal rates.
* ASPA processing latency, when observable.
* BGP route-processing throughput.
* Route revalidation time, when observable.
* BGP convergence time, when applicable.
* CPU and memory utilization.
* Processing backlog and failures.

**Reporting:**

The report MUST identify the timing relationship between ASPA and BGP updates.

Results SHOULD compare concurrent workload performance with the corresponding isolated workload measurements.

## Resource Utilization

**Objective:**

Characterize the CPU and memory resources associated with enabling ASPA verification under different workload conditions.

This test evaluates steady-state resource consumption and transient resource utilization during ASPA synchronization, verification, revalidation, and updates.

**Test Conditions:**

The Measurement System MUST provide resource-utilization measurements with a documented sampling interval.

Where available, resource measurements SHOULD distinguish total system utilization from utilization of the processes responsible for BGP and ASPA processing.

**Procedure:**

1. Configure the DUT with ASPA verification disabled.

2. Establish a predefined BGP route population and reach steady state.

3. Record baseline CPU and memory utilization.

4. Enable ASPA verification and load the selected ASPA dataset.

5. Establish an equivalent BGP workload and reach steady state.

6. Record CPU and memory utilization with ASPA verification enabled.

7. Execute the selected ASPA-related workloads, including:

   * Initial ASPA synchronization.
   * BGP-triggered AS_PATH verification.
   * Incremental ASPA updates.
   * ASPA-triggered route revalidation.
   * Sustained ASPA churn.

8. Record resource utilization throughout each workload.

9. Restore the initial conditions and repeat using different ASPA dataset sizes and BGP workload characteristics.

The tester SHOULD measure steady-state and transient resource utilization separately.

For comparisons between ASPA-enabled and ASPA-disabled configurations, other relevant configuration parameters and the BGP workload MUST remain equivalent.

**Measurements:**

* Baseline CPU utilization.
* CPU utilization with ASPA verification enabled.
* Baseline memory consumption.
* Memory consumption with ASPA verification enabled.
* Peak CPU utilization.
* Peak memory consumption.
* Per-process resource utilization, when available.

The additional steady-state memory consumption MAY be calculated as:

    Delta_M = M_ASPA - M_Baseline

where M_ASPA and M_Baseline are measured under equivalent BGP workload conditions.

This difference characterizes the additional memory associated with the ASPA-enabled configuration. It MUST NOT be interpreted as the memory occupied exclusively by ASPA records.

**Reporting:**

The report MUST specify the measurement source, sampling interval, and scope of resource-utilization measurements.

Results SHOULD include steady-state resource consumption, peak resource utilization, and changes associated with increasing ASPA dataset or BGP workload size.

## RTR Session and Recovery Behavior

**Objective:**

Characterize ASPA verification behavior and recovery performance following RTR session interruptions, cache resets, and cache failover.

Recovery behavior depends on the applicable RTR protocol procedures and timing parameters, including the Refresh, Retry, and Expire intervals.

These tests MUST use recovery scenarios consistent with the applicable RTR protocol specification.

### RTR Session Interruption and Reconnection

This test evaluates DUT recovery following a temporary RTR session interruption.

**Test Conditions:**

The DUT MUST initially contain a predefined ASPA dataset and BGP route population.

The RTR session MUST be established, and the expected verification states MUST be confirmed before the interruption.

**Procedure:**

1. Establish the RTR session and load the baseline ASPA dataset.

2. Establish the baseline BGP route population.

3. Confirm the expected verification states.

4. Record the configured RTR timing parameters.

5. Establish steady state.

6. Trigger a controlled RTR session interruption.

7. Record the interruption time.

8. Observe whether previously received ASPA information remains available during the interruption.

9. Restore RTR connectivity after a predefined interval.

10. Measure the time required to re-establish the RTR session and complete any necessary synchronization.

11. Determine when the expected verification and routing states have been restored.

12. Record resource utilization and recovery failures.

13. Restore the baseline conditions and repeat.

The tester SHOULD distinguish interruptions shorter than the configured Expire Interval from interruptions that cause the previously received data to expire.

The benchmark MUST account for the applicable RTR data-retention and expiration requirements.

**Measurements:**

* RTR session reconnection time.
* ASPA resynchronization time.
* Time to restore the expected verification state, when observable.
* Externally observable routing-state recovery time, when applicable.
* CPU and memory utilization.
* Recovery failures.

**Reporting:**

The report MUST include the interruption duration, RTR timing parameters, and observed ASPA data-retention behavior.

### Cache Reset and Full ASPA Resynchronization

This test evaluates the DUT when the RTR cache cannot provide the requested incremental update and a full synchronization is required.

**Procedure:**

1. Establish a stable RTR session and load the baseline ASPA dataset.

2. Establish the baseline BGP route population.

3. Confirm the expected verification and routing states.

4. Configure the RTR Cache/Emulator so that the DUT requests an incremental update that the cache cannot provide.

5. Cause the cache to respond with a Cache Reset PDU according to the applicable RTR protocol procedures.

6. Observe the DUT's recovery behavior and subsequent RTR exchanges.

7. If the DUT proceeds with full synchronization, record the corresponding Reset Query, Cache Response, payload PDUs, and End of Data PDU.

8. Measure the time required to complete the RTR synchronization.

9. Determine when the expected ASPA verification and routing states have been restored, using the available observation mechanisms.

10. Record resource utilization and recovery failures.

11. Restore the baseline conditions and repeat.

The tester SHOULD evaluate scenarios in which the restored ASPA dataset is unchanged and scenarios in which it differs from the original dataset.

When the dataset changes, the expected verification-state transitions MUST be determined before testing.

A Cache Reset PDU does not, by itself, imply immediate invalidation or removal of all previously received ASPA information. The DUT's handling of retained data and subsequent recovery MUST be interpreted according to the applicable RTR specification.

**Measurements:**

* Cache reset recovery time.
* Full RTR data transfer time.
* ASPA resynchronization time, when observable.
* Route revalidation time, when observable.
* BGP convergence time, when applicable.
* CPU and memory utilization.
* Recovery failures.

**Reporting:**

The report MUST identify the triggering condition, RTR protocol exchanges, ASPA dataset characteristics, and observation points used to determine recovery completion.

### RTR Cache Failover

This optional test evaluates DUT behavior when ASPA information is obtained from an alternative RTR cache following the unavailability of a previously used cache.

**Test Conditions:**

The test environment MUST provide at least two RTR caches or emulators.

The datasets and configurations of the caches MUST be documented.

The DUT MUST support a cache configuration that allows the selected failover scenario to be exercised.

**Procedure:**

1. Configure the DUT with multiple RTR caches according to its supported cache selection behavior.

2. Establish the baseline ASPA dataset and BGP route population.

3. Confirm the expected verification and routing states.

4. Establish steady state.

5. Trigger a controlled failure of the cache selected for the test.

6. Observe the DUT's recovery behavior and RTR protocol exchanges.

7. Determine whether and when the DUT begins obtaining ASPA information from an alternative cache.

8. Measure the time required to restore the expected ASPA verification state, when observable.

9. Record any associated route revalidation or BGP routing-state changes.

10. Restore the baseline conditions and repeat.

The tester SHOULD distinguish between scenarios in which the caches contain equivalent ASPA datasets and scenarios in which their datasets differ.

The test MUST account for the DUT's supported cache selection behavior and the applicable RTR protocol semantics.

**Measurements:**

* RTR recovery time.
* ASPA resynchronization time.
* Time to restore the expected verification state, when observable.
* Externally observable routing-state recovery time, when applicable.
* CPU and memory utilization.
* Recovery failures.

**Reporting:**

The report MUST identify the cache configuration, dataset characteristics, relevant timing parameters, and observed recovery behavior.

If the DUT does not support the selected cache failover scenario, the test MUST be reported as not applicable rather than as a performance failure.

# Reporting Requirements 

Benchmarking results MUST be reported with sufficient information to support repeatability, reproducibility, and meaningful comparisons across different ASPA verification implementations.

The benchmarking report MUST document the test environment, DUT configuration, workload characteristics, measurement methods, and results, as applicable to the tests performed.

The following information SHOULD be included:

* **Test Environment:** DUT hardware and software specifications, test topology, RTR Cache/Emulator, BGP Route Generator, and measurement tools.

* **DUT Configuration:** ASPA verification settings, BGP neighbor relationships, verification directions, routing policies, and relevant RTR parameters.

* **ASPA Dataset Characteristics:** ASPA record count, provider authorization count, provider-set cardinality distribution, ASPA coverage, and ASPA update characteristics.

* **BGP Workload Characteristics:** BGP route count, unique AS_PATH count, AS_PATH length distribution, verification outcome distribution, and BGP update characteristics.

* **Measurement Methods:** Measurement metrics, observation points, timing methods, and any limitations of the instrumentation. Measurements obtained through internal instrumentation MUST be distinguished from externally observable measurements.

* **Benchmark Results:** Measured latency, throughput, convergence time, scalability, and resource utilization, as applicable. The number of test repetitions and relevant statistical information SHOULD also be reported.

* **Test Limitations:** Unsuccessful tests, unexpected verification results, measurement limitations, and deviations from the specified procedures.

For benchmarks involving ASPA-triggered route revalidation, the report SHOULD additionally include the number of affected routes, the number of distinct affected AS_PATHs, and the corresponding verification-state transitions.

Results from different implementations SHOULD be compared only when the relevant workload characteristics, measurement definitions, and test conditions are equivalent.

The report SHOULD clearly identify any measurements that could not be obtained and distinguish them from unsuccessful test results.

# Security Considerations

This document defines a benchmarking methodology for ASPA-based AS_PATH verification and does not introduce new protocols, verification algorithms, or routing security mechanisms. The security considerations associated with ASPA verification are discussed in {{I-D.ietf-sidrops-aspa-verification}}.

The benchmarking procedures described in this document are intended for isolated laboratory environments, consistent with {{RFC6815}}. Tests involving large ASPA datasets, high BGP update rates, ASPA churn, or repeated RTR session interruptions may impose substantial processing and resource demands on routing devices. Such tests MUST NOT be conducted on production networks carrying live user traffic.

Synthetic ASPA datasets and BGP routing information used during benchmarking MUST NOT be propagated into operational RPKI infrastructure or production routing systems. The test environment SHOULD be configured to prevent unintended interactions with external networks.
The routing policies applied to ASPA verification results may affect route selection and convergence behavior. These policies MUST be documented to ensure that benchmark results are interpreted correctly.

This document focuses on performance characterization rather than the security effectiveness or protocol conformance of ASPA verification implementations. Benchmarking results alone do not establish the security properties of an implementation.


# IANA Considerations

This document has no IANA actions.

--- back