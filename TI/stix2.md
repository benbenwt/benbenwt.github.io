- SDO

  攻击者的意图，前期活动，部署恶意软件执行。
  我们只能对软件进行分析，活动和意图无法复现。
  ​
  ​分析软件提取：静态分析得到（攻击链，类型），动态分析
  一手检测：攻击者，组织，设施。不然只能爬取别人的数据。
  高级别抽象的的数据：目标，动机
  ​还有分析的工具啥的，直接填。

  - attack pattern

    ttp：Tactics、技术Techniques和过程Procedures

    - external references
      "external_references": [              
        {                    "source_name": "capec",                    
      "url": "https://capec.mitre.org/data/definitions/148.html",               
           "external_id": "CAPEC-148"                }        
          ]
      使用capec规定的pattern，可从微步得到

    - name
      并非指attackpattern

    - kill_chain_phases
      ida

    - creating_by_ref
      攻击者

    - object_marking_refs
      marking_definition:决定数据分享标准
      {          "type": "marking-definition",  
              "spec_version": "2.1",        
        "id": "marking-definition--f88d31f6-486f-44da-b317-01333bde0b82",          
      "created": "2017-01-20T00:00:00.000Z",       
         "definition_type": "tlp",      
          "definition": {              "tlp": "amber"          }   
         },
      ​用于与正确的受众共享信息

    - 非内嵌关系

      - delivers  malware

      - targets identity,location,vulnerability

      - uses

  - compaign

    - name，description

    - first_seen,last_seen,objective（目标）

  - course of  action
    - action...

  - grouping

  - identity

  - indicator

  - infrastructure

    - infrastructure_types
      infrastructure-type-ov设施作用，如放大，干扰等

    - kill_chain_phases

  - Intrusion set

    - goals
      ?

    - fitst_seen,last_seen

    - resource_level
      value came from vocabulary attack-resoutce-level-ov
      ​个人，队伍，组织等。

    - motivation...

    - 固定内嵌

    - 其他关系
      attributed-to threat-actor
      compromises infrastructure
      hosts,owns  infrastructure
      originates-from location
      ​

  - location

    - name，des

    - latitude，longitude

    - precision精度

    - region

    - country

    - administrative_area

    - city

    - street_address

    - postal_code

  - malware

  - malware analysis

    static or dynamic analysis

    - result

    - analysis_sco_refs

    - product用啥分析的

    - version工具版本

    - host_vm_ref 动态分析的虚拟环境

    - installed_software_refs

    - configuration——version？

    - modules

    - 

  - Note

    othrer information

    - content

    - abstract

    - authors

    - object_refs

  - observed data

    cyber security reslted entities,files ,systems,networks.
    raw information

    - object_ref,list of sco and sros representing the observation

    - first_seen,last_seen

  - Opinion

    - explanation，why

    - authors

    - opinion，

    - object——refs

  - report

    description of more topics,such as threat actor,malware..
    连接相关连的物体。

    - name,description

  - Threat Actor

    - name

    - goals

    - sophistication

  - Tool
    - kill_chains

  - vulnerability

    - external_references

    - name

  - 

- SRO

- SCO

- VO

- 其他

  - import url
    stix api documents:https://docs.oasis-open.org/cti/stix/v2.1/cs01/stix-v2.1-cs01.html#_pcpvfz4ik6d6
    stix example:http://stixproject.github.io/documentation/idioms/file-hash-reputation/

  - stix 2.1 differs from stix2.0 in the following ways
    new objects:Grouping,infrastructure,Language-Content(internationnlization),Location,Malware-Analysis,Note,Opinion