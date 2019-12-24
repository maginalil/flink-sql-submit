# Flink SQL Submit

Currently, Flink 1.9 doesn't support to submit a DDL statement in SQL CLI, and also doesn't support to submit a SQL script.
This is not convenient for walkthrough and demo. That's why I created a such a project.

## How to use

1. Set your Flink and Kafka install path in `env.sh`
2. Add your SQL scripts under `src/main/resources/` with `.sql` suffix, e.g, `q1.sql`
3. Start all the service your job needed, including Flink, Kafka, and DataBases.
3. Run your SQL via `./run.sh <sql-file-name>`, e.g. `./run.sh q1`
4. If the terminal returns the following output, it means the job is submitted successfully.

```
Starting execution of program
Job has been submitted with JobID d01b04d7c8f8a90798d6400462718743
```



按照上述步骤无法执行成功.需要修改的地方如下
1.缺少mysql的建表语句.
CREATE TABLE `pvuv_sink` (
  `pv` bigint(20) DEFAULT NULL,
  `uv` bigint(20) DEFAULT NULL,
  `dt` datetime DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
2.flink使用1.9.1(1.9.0应该也可以)

3.下载好依赖包(全部从这个版本里下载). https://ci.apache.org/projects/flink/flink-docs-release-1.9/dev/table/connect.html   
flink-connector-kafka 使用0.11+ (universal)这个   具体参考sql文件或者yaml文件中kafka的version写的什么. 用别的版本会失败
flink-jdbc
flink-json

4.修改run.sh  我执行的时候无法加载flink_home/lib下的包.暂时不知道为啥,手动把他们加载上(或者试试把所有依赖打进jar包里?)
$FLINK_DIR/bin/flink run --classpath file:///data/magina/flink-1.9.1/lib/flink-dist_2.11-1.9.1.jar --classpath file:///data/magina/flink-1.9.1/lib/flink-jdbc_2.11-1.9.0.jar --classpath file:///data/magina/flink-1.9.1/lib/flink-json-1.9.0-sql-jar.jar --classpath file:///data/magina/flink-1.9.1/lib/flink-sql-connector-kafka_2.11-1.9.0.jar --classpath file:///data/magina/flink-1.9.1/lib/flink-table_2.11-1.9.1.jar --classpath file:///data/magina/flink-1.9.1/lib/flink-table-blink_2.11-1.9.1.jar --classpath file:///data/magina/flink-1.9.1/lib/log4j-1.2.17.jar --classpath file:///data/magina/flink-1.9.1/lib/mysql-connector-java-5.1.8.jar --classpath file:///data/magina/flink-1.9.1/lib/slf4j-log4j12-1.7.15.jar  -d -p 4 target/flink-sql-submit.jar -w /data/magina/test/src/main/resources/ -f "$1".sql

5.启动source-generator.sh 向kafka写数据. 然后等着看mysql的数据更新吧.
PS: 我某台服务器无法启动taskmanager. 手动修改taskmanager.sh中的 #TM_MAX_OFFHEAP_SIZE="8388607T" 为TM_MAX_OFFHEAP_SIZE="1024M"(暂时不清楚情况)

