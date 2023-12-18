# HiveOverYt
## Исследование в рамках проектного курса в ШАДе (осень 2023)

### Исполнители: Попов Даниил, Ермолаев Илья
HADOOP - один из самых популярных фреймворков для обработки больших данных. Основными компонентами которого являются:
* HDFS (Hadoop Distributed File System) - предназначенная для хранения файлов больших размеров
* Движки для вычислений в рамках парадигмы MapReduce, такие как Hadoop MapReduce или YARN.

У Яндекса есть свой аналог Apache Hadoop – YT, с недавнего времени YTsaurus. В рамках проектного курса мы ПЫТАЛИСЬ научить движок Apache Hive работать с YTsaurus, подменив в нём запуск операций в hadoop, на запуск операций в YTsaurus.

Исходники:
[YTsaurus](https://github.com/ytsaurus/ytsaurus), [Hive](https://github.com/apache/hive), [Hadoop](https://github.com/apache/hadoop).

Hive работает поверх HDFS и запускает Map / Reduce операции на hadoop кластере.

[QLDriver](https://github.com/apache/hive/blob/aa0237d62099d23bcfadb1ff4c4171a15de25447/ql/src/java/org/apache/hadoop/hive/ql/Driver.java#L141) принимает SQL-Like команду и строит из нее [QueryPlan](https://github.com/apache/hive/blob/aa0237d62099d23bcfadb1ff4c4171a15de25447/ql/src/java/org/apache/hadoop/hive/ql/Driver.java#L519) - граф выполнения операции. Внутри нее  происходит парсинг и оптимизация и записывается в DriverContext.
Далее происходит запуск [Executor](https://github.com/apache/hive/blob/aa0237d62099d23bcfadb1ff4c4171a15de25447/ql/src/java/org/apache/hadoop/hive/ql/Driver.java#L363), который запускает [выполнение](https://github.com/apache/hive/blob/master/ql/src/java/org/apache/hadoop/hive/ql/Executor.java#L85) QueryPlan.
Происходит предобработка и корневые [Task](https://github.com/apache/hive/blob/master/ql/src/java/org/apache/hadoop/hive/ql/exec/Task.java#L55)'и [кладутся](https://github.com/apache/hive/blob/master/ql/src/java/org/apache/hadoop/hive/ql/Executor.java#L177) в Executor.
Далее в [цикле](https://github.com/apache/hive/blob/master/ql/src/java/org/apache/hadoop/hive/ql/Executor.java#L243) происходит обработка тасок. 
Берется и очередная таска и исполняется в [TaskRunner](https://github.com/apache/hive/blob/master/ql/src/java/org/apache/hadoop/hive/ql/Executor.java#L346C5-L346C58).
Если это MapReduce операция она исполняется в [ExecDriver](https://github.com/apache/hive/blob/master/ql/src/java/org/apache/hadoop/hive/ql/exec/mr/ExecDriver.java).

/**
 * ExecDriver is the central class in co-ordinating execution of any map-reduce task.
 * It's main responsibilities are:
 *
 * - Converting the plan (MapredWork) into a MR Job (JobConf)
 * - Submitting a MR job to the cluster via JobClient and ExecHelper
 * - Executing MR job in local execution mode (where applicable)
 *
 */

[MapredWork](https://github.com/apache/hive/blob/master/ql/src/java/org/apache/hadoop/hive/ql/plan/MapredWork.java#L36) - класс в котором хранится описание [MapWork](https://github.com/apache/hive/blob/master/ql/src/java/org/apache/hadoop/hive/ql/plan/MapWork.java#L75) и [ReduceWork](https://github.com/apache/hive/blob/master/ql/src/java/org/apache/hadoop/hive/ql/plan/ReduceWork.java#L49)

Они содержат описание части Map или Reduce операции соответсвенно.
* Временные таблицы
* Входные таблицы
* Дерево операторов
* Конфиг партицирования данных и другое
* Количество тасок

Hadoop исполняет операции согласно описанию в JobConfig. ExecDriver "накачивает" его своими кастомными классами:

[ExecMapRunner](https://github.com/apache/hive/blob/master/ql/src/java/org/apache/hadoop/hive/ql/exec/mr/ExecMapRunner.java#L29) - запускает выполнение Map операции </br>
[ExecMapper](https://github.com/apache/hive/blob/master/ql/src/java/org/apache/hadoop/hive/ql/exec/mr/ExecMapper.java#L61) - непосредственно класс Map операции. Метод map преобразует входную строку, применяя цепочку операторов

По аналогии работают классы [ExecReduceRunner]() и [ExecReducer]() 
Также в методе ExecDriver'a происходит создание временных таблиц для будущих операций. 
Затем Создается [JobClient]() в который передается "надутый" [JobConfig](), где собственно и исполняется





