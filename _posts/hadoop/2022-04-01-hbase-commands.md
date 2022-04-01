---
title:  "[HBase] HBase Commands 모음"

categories:
  - HBase
tags:
  - [HBase, NoSQL, hadoop]
toc: true
toc_sticky: true
---

* HBase Shell commands 

1) General HBase shell commands 

- status 
    : cluster 상태를 보여주며, 추가옵션을 통해 상세정보를 확인

> status 
> status 'simple' 
> status 'summary'
> status 'detailed' 

	

- version
    : 설치된 HBase의 버전정보를 확인

> version 



- whoami 
    : 현재 hbase의 사용자를 확인

> whoami 

		

2) Table Management commands 

- alter
    : (1)table의 schema를 변경하기위해 사용된다. dictionary 형태로 나열한다. 

> alter 'table1' , NAME>= 'cf1', VERSIONS>= 5 
//  'table1'의 'cf1'의 version 정보가 5까지 저장되도록 생성/변경한다. 


> alter 'table1', 'cf1',  {NAME>='cf2', IN_MEMORY>=true}, {NAME=>'cf3', VERSIONS >=5}
// 'table1' 에 'cf1'을 생성/변경 하고 'IN_MEMORY' 옵션이 true인 'cf2'을 생성/변경하고,
'VERSIONS' 옵션이 5인 'cf3'을 생성/변경한다. 

			

: (2)column family를 삭제한다. 

> alter 'table1', 'delete'>='cf1'
// 'table1'의 'f1' column family를 삭제한다. 


: (3)region 설정을 변경한다. 

> alter 'table1', MAX_FILESIZE => '123217728' 
// region의 최대 크기를 128 MB로 변경한다.		

			

: (4)coprocessor를 설정한다. table에 대해서 coprocessor를 설정할 수 있으며 자동으로 시퀀스가 유니크하게 추가 된다. 
 coprocessor attribute는 F/W이 coprocessor class를 인지할 수 있도록 다음과 같은 패턴에 맞아야 한다. 
- [coprocessor jar file location] | class name | [priority] | [arguments]

> alter 'table1', 'coprocessor'=>'hdfs:///foo.jar|com.foo.FooRegionObserver|1001|arg1=1,ar...'
// table coprocessor를 설정한다. 


:(5) colum family에 특정한 설정을 추가한다. 
	> alter 'table1', CONFIGURATION => {'hbase.hregion.scan.loadColumnFamiliesOnDemand' => 'true' } 
	> alter 'table1', {NAME=> 'cf2', CONFIGURATION => {'hbase.hstore.blockingStoreFiles' => '10'}}	
        // column family 'cf2'의 Hfile(storeFile) 갯수를 10개로 설정한다. 



: (6) table-scope 설정을 삭제할 수도 있다. 
	> alter 'table1', METHOD => 'table_att_unset', NAME => 'MAX_FILESIZE'  // region 최대 크기 설정을 해제한다.
	> alter 'table1', METHOD => 'table_att_unset', NAME => 'coprocessor$1' // coprocessor 설정을 해제한다. 

		

	:(7) 여러 설정을 동시에 수행하는 것도 가능하다. 
        	> alter 'table1', { NAME => 'cf1', VERSIONS => 3 }, { MAX_FILESIZE => '134217728' }, 
                { METHOD => 'delete', NAME => 'cf2'}, OWNER => 'johndoe', METADATA => { 'mykey' => 'myvalue' }

		

- create
    : 테이블을 생성하며 옵션을 추가한다.

> create 'table1', {NAME => 'cf1', VERSIONS => 5}	
    // table1을 생성하면서 최신의 version을 5개까지 저장하는 column family cf1을 생성한다. 

> create 'table1', {NAME => 'cf1', NAME => 'cf2', NAME => 'cf3'}
// table1을 생성하면서 column family cf1, cf2, cf3 을 생성한다. 

> create 'table1', {NAME => 'cf2', VERSION =>1, TTL => 2592000, BLOCKCACHE =>true} 
> create 'table1', {NAME => 'cf2', CONFIGURATION => {'hbase.hstore.blockingStoreFiles' ='10'}} 

		

- describe 
    : table의 정보를 보여준다. 

> describe 'table1' 

	

- enable /disable
    : table을 enable / disable 상태로 변경한다. 

		> enable 'table1' 
		> disable 'table1'

	

- enable_all / disable_all 
    : 다음과 같은 정규식 규칙에 맞는 테이블에 대해 모두 enable / disable 시킨다. 

> enable_all 'table_*'
> disable_all 'table_*'



- is_enabled / is_disabled
    : enabled / disabled 상태인지 확인한다. 
	> is_disabled 'table1' 

		

- drop
    : table을 drop 시킨다. 반드시 먼저 disable을 시켜야 한다. 
	> drop 'table1' 

		

- drop_all
    : 다음과 같은 정규식 규칙에 맞는 테이블을 모두 drop 한다. 
	> drop_all 'table1'

	

- exists
    : table이 있는지 조회한다.
        > exists 'table1'

	

- list
    : hbase 의 모든 table 리스트를 출력한다. 추가적으로 정규식을 사용할 수도 있다. 
	> list	
	> list 'abc.*' 

		

- show_filters
     : hbase의 모든 filters 들을 보여준다. 
	> show_filters 

		

- alter_status
    : alter 명령어의 상태를 보여준다. schema가 변경되고 있는 table들의 이름과 region 수를 보여준다 
	> alter_status 'table1' 

		

- alter_async
    : 비동기적으로 alter 명령을 수행하여 schema 변경에 대한 결과를 기다리지 않는다. 
	> alter_async 't1', NAME => 'f1', METHOD => 'delete'or a shorter version:hbase> alter_async 't1', 'delete' => 'f1'

	



3) Data Manipulation commands  

- count
    : table의 row 수를 출력한다. 이 명령어는 오랜 시간동안 수행되는데, mapreduce job을 통해서 수행되기 때문이다. 
    row의 수는 매 1000 번째 마다 출력되는데, 이러한 설정은 옵션을 통해 변경 가능하다. 
        > count 'table1', INTERVAL => 100000 
	> count 'table2', CACHE => 1000
	> count 'table3', INTERVAL >= 10, CACHE =>1000
	> scan 't1', {RAW => true, VERSIONS => 10}

		

: 똑같은 명령어를 table의 레퍼런스를 사용하여 실행할 수도 있다. 
    > table1.count INTERVAL => 100000 
    > table1.count CACHE => 1000 
    > table1.count INTERVAL => 10, CACHE => 1000 

		

- delete
    : 특정 셀의 값을 삭제한다. 반드시 cell의 값과 동일하게 옵션을 입력해야 삭제가 된다. 동일하게, table 레퍼런스로도 삭제가 가능하다. 
	> delete 'table1', 'rowkey1', 'cellvalue1', timestamp#
	> table1.delete 'rowkey1', 'cellvalue1', timestamp#

		

- deleteall
    : 주어진 row의 모든 cell의 값을 삭제한다. 동일하게, table 레퍼런스로도 삭제가 가능하다. 
	> deleteall 'table1', 'rowkey1' 
	> deleteall 'table1', 'rowkey1', 'cellvalue1', timestamp#
	> table1.deleteall 'rowkey1', 'cellvalue1'
	> table1.deleteall 'rowkey1', 'cellvalue1', timestamp#

		

- get
    : 특정 row나 cell의 값을 가져올 때 사용한다. table 명, row와 기타 옵션을 추가할 수 있다.
    table의 레퍼런스로도 동일하게 값을 가져올 수 있다. 
	> get 'table1', 'rowkey1' 
	> get 'table1', 'rowkey1', {TIMERANGE => [start_timestamp, end_timestamp]}
	> get 'table1', 'rowkey1', {COLUMN => 'cf1', TIMESTAMP => ts1}
	> get 'table1', 'rowkey1', {COLUMN => ['cf1', 'cf2', 'cf3'] , VERSIONS =>4 }
	> get 'table1', 'rowkey1', {FILTER => "ValueFilter (=, 'binary:abc')"}
	> table1.get 'rowkey1', 'cf1', 'cf2'
	> table1.get 'rowkey2', {COLUMN => 'cf1', TIMESTAMP => ts2, VERSIONS =>4 }
	> table1.get 'rowkey1', {FILTER => "ValueFilter(=, 'binary:abc')"}		

:기본적으로 'toStringBinary' 포맷임으로 get은 커스텀 포맷을 지원한다. 사용자는 FORMATTER를 column name을 사용하여 정의할 수 있다. 	하지만 모든 column family의 column에 대해서는 포맷을 정의할 수 없다. 
    > get 'table1', 'rowkey1', {COLUMN => ['cf1:qualifier1:tolInt', 'cf2:qualifier2:c(org.apache.hadoop.hbase.util.Bytes).toInt']} 

		

- get_counter
    : table, row, column 에 대한 cell의 수를 리턴한다. 
	> get_counter 'table1', 'rowkey1', 'cf1'
	> table1.get_counter 'rowkey1', 'cf2' 

- incr
    : 특정한 cell의 값을 증가시킨다. table 레퍼런스로도 동일하게 값을 증가시킬 수 있다. 
	> incr 'table1', 'rowkey1', 'cf1' 
	> incr 'table1', 'rowkey2', 'cf2', 1 
	> incr 'table1', 'rowkey3', 'cf3', 10
	> table1.incr 'rowkey1', 'cf1', 1 
	> table2.incr 'rowkey2', 'cf2', 10 		

- put
    : cell의 값을 입력한다. 
	> put 'table1', 'rowkey1', 'cf1', 'value', ts1 
	> t.put 'rowkey1', 'cf1', 'value', ts1 		

 - scan
    : 테이블의 값을 조회한다 (=select). Scanner는 TIMERANGE, FILTER, LIMIT, STARTROW, STOPROW, TIMESTAMP, MAXLENGTH 이나
    COLUMNS, CACHE 등의 옵션을 추가할 수 있다.기본적으로 'toStringBinary' 포맷이나 scan 명령어는 커스텀 포맷을 지원한다. 
     column family의 모든 값들을 조회하고자 할때는 qualifier를 비워두면 된다(ex. 'cf1:').  필터는 두 가지 방법으로 설정할 수 있다.      

	1) fiterString 사용 (hbase-4176) 

	2) filter의 전체 package 명 사용 
		> scan '.META.' 
		> scan '.META.', {COLUMNS => 'info:regioninfo'}
		> scan 'table1', {COLUMNS => ['cf1', 'cf2'], LIMIT =>10, STARROW =>'xyz'}
		> scan 'table1', {COLUMNS => 'c1', TIMERANGE => [ts1, ts2]}
		> scan 'table1', {FILTER => "(PrefixFilter ('row2') AND (QualifierFilter (>=, 'binary:xyz')))" AND (TimestampsFilter (123, 456))}
		> scan 'table1', {COLUMN => ['cf1:qualifier1:tolInt', 'cf2:qualifier2:c(org.apache.hadoop.hbase.util.Bytes).toInt']} 
		> scan 'table1', {FILTER => org.apache.hadoop.hbase.filter.ColumnPaginationFilter.new(1,0)} 
		> scan 'table1', {COLUMNS => ['cf1', 'cf2'], CACHE_BLOCKS >=false}
		> scan 'table1', {RAW => true, VERSIONS =>10}
                // RAW 명령어는 delete marker가 표시된 것, 삭제되었지만 집계되지 않은 것을 포함하여 모든 cell의 값들을 가져온다. 

		> t = get_table 'table1'
		> t.scan 

			

- truncate
    : table을 disable, drop 후 재 생성한다. 
	>truncate 'table1' 





4) HBase surgery tools

- assign
    : region을 assign한다. 만약 region이 이미 할당되어 있다면 이 명령어를 통해서 강제로 재할당이 이루어진다. 
	> assign 'REGION_NAME'

		

- balancer
    : 클러스터 balancer를 트리거 한다. balancer가 할당되지 않은 모든 region들을 밸런싱 할 수 있도록 전달되었으면
    true를 리턴하고, region 이동이 발생하지 않으면 false가 출력된다. 
	> balancer 

		

-  balancer_switch true/false
    : balancer 를 활성화/비활성화 시킨다. 이전의 balancer 상태를 리턴한다. 
	> balance_switch true
	> balance_switch false



- close_region 
: 하나의 region을 닫는다. hmaster에게 region을 클러스터에서 닫을 것을 요청하거나 만약에 특정 서버명이 주어졌다면 해당 regionserver에서 region을 직접 제외하도록 요청한다. region을 닫으면서 hmaster는 region명을 확인한다. regionserver에 직접     region을 닫을 것을 요청하는 경우 region의 암호화된 코드만 입력하면 된다. 

region의 이름은 TestTable, 0094429456, 1289497600452.527db22f95c8a9e0116f0cc13c68'과 같이 있는데 뒷부분인 527db22f95c8a9e0116f0cc13c68' 만 입력해주면 된다. 암호화된 region명은 region이름의 해시값이다. 서버명과 같이 port와 startcode로 를 전달하는 경우는 해당 regionserver에서 region을 닫게 된다. 

> close_region '527db22f95c8a9e0116f0cc13c68'
> close_region '527db22f95c8a9e0116f0cc13c68', 'host187.example.com,60020,1289493121758'

	

- compact
    :  전달된 모든 table의 region들을 comaction을 수행한다.  region 하나씩 comaction 수행 가능하다. 
	> compact 'table1' // 해당 table의 모든 region을 comaction을 한다. 
	> compact 'region1' // 전체 region을 comaction을 수행한다. 
	> compact 'region1', 'cf1' // 해당 region의 column family만 comaction을 수행 한다. 
	> compact 'table1', 'cf1' // 해당 테이블의 column family만 comaction을 수행한다. 		

- flush
    : table의 모든 region나, 하나의 region에 대해서 flush(memstore to storefile) 를 수행한다. 
	> flush 'table1'
	> flush 'region1'

		

- major_compact
    :  특정 table에 대해서 major comaction을 수행하거나 개별 region에 대해서 major compaction 수행한다. 
	> major_compact 'table1'
	> major_compact 'table1', 'cf1'
	> major_compact 'region1'

- move
    : region을 이동시킨다. 옵션으로 타겟 regionserver를 설정할 수 있으며 따로 설정을 하지 않으면 랜덤으로 이동한다.
    encoded region name과 서버명으로 host,port, startcode를 같이 사용한다. 
	> move '527db22f95c8a9e0116f0cc13c680396' 
	> move '527db22f95c8a9e0116f0cc13c680396' 'host187.example.com,60020,1289493121758'

		

- splite
    : 전체 table을 split 하거나 하나의 region을 split 수행한다. 추가적인 파라미터를 설정할 수 있는데 region의 split key를 명시할 수 있다.
	> split 'table1'
	> split 'region1', 
	> split ‘regionName' # format: ‘tableName,startKey,id'
	> split ‘tableName', ‘splitKey'
	> split ‘regionName', ‘splitKey'
		

- unassign
    : region을 할당 해제한다. 할당 해제된 region은 위치하고 있는 regionserver에서 close 상태로 변경되고 다시 open 상태로 변경된다. 
    만약 true 옵션과 같이 수행하면 재할당 하기 전에 hmaster 안의 in-memory 상태를 제거한다. 만약 중복으로 assign 되어 있다면
    hbck -fix를 수행하여 해결해야 하며, 해당 작업은 신중하게 해야 한다. critical한 상황이 아니거든 사용하지 않는 것이 좋다. 
	> unassign 'myregion1' 
	> unassign 'myregion1', true

		

- hlog_roll
    : log writer를 새로운 파일에 쓸수 있도록 롤링작업을 수행한다. 
	>hlog_roll 

	

- zk_dump
    : zookeeper 가 바라보는 hbase 클러스터 상태를 확인한다. 
	>zk_dump



		

5) Cluster replication tools

- add_peer
    : 복제replication를 위한 짝peer을 추가한다. id는 반드시 짧아야 하고 cluster key가 다음과 같이 구성되어야 한다.
      'hbase.zookeeper.quorum:hbase.zookeeper.property.clientPort:zookeeper.znode.parent')


       	> add_peer '1', "server1.cie.com:2181:/hbase"
	> add_peer '2', "zk1,zk2,zk3:2182:/hbase-prod"



- remove_peer
    : replication stream을 중지하고 그것과 관련된 메타정보를 삭제한다. 
	> remove_peer '1'

- list_peers 
    : cluster 상의 모든 replication peer를 확인한다. 
	> list_peers

	

- enable_peer/disable_peer
    : 특정 peer로의 복제작업을 시작/중지시킨다. 
	> enable_peer '1'
        > disable_peer '1'


- start_replication/stop_replication 
    : 등록된 모든 replication을 (재)시작 / 중지한다. 매우 긴급한 상황이 아니고서 쓰지 않는 명령이다. 	
        > start_replication
	> stop_replication 



6) Security tools

- grant
    : user들에게 특정 권한을 부여한다. 'RWXCA'을 사용하여 권한부여 가능하다(READ(‘R’), WRITE(‘W’), EXEC(‘X’), CREATE(‘C’), ADMIN(‘A’)).
	> grant 'paul', 'RWXCA'
	> grant 'parker', 'RW' 'table1', 'cf1', 'qualifier1'

		

- revoke
    : 사용자의 접근 권한을 삭제한다. 
	> revoke 'parker', 'table1', 'cf1', 'qualifier1'

	

- user_permission
    : 특정 테이블에 존재하는 모든 권한을 보여준다. 
	> user_permission 'table1' 