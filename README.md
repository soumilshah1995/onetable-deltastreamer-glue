# onetable-deltastreamer-glue
onetable-deltastreamer-glue
![Screenshot 2024-03-09 at 5 07 30 PM](https://github.com/soumilshah1995/onetable-deltastreamer-glue/assets/39345855/90e8ef71-5a5f-4ced-bf5a-045411a7d0c9)


# Download Jar and Sample dataset 
* https://drive.google.com/drive/folders/1mQzZSVgxQGksoXkYGb8DSn2jeR48VehS?usp=share_link


# Delta Streamer job 
```

import com.amazonaws.services.glue.GlueContext
import com.amazonaws.services.glue.MappingSpec
import com.amazonaws.services.glue.errors.CallSite
import com.amazonaws.services.glue.util.GlueArgParser
import com.amazonaws.services.glue.util.Job
import com.amazonaws.services.glue.util.JsonOptions
import org.apache.spark.SparkContext
import scala.collection.JavaConverters._
import org.apache.spark.sql.SparkSession
import org.apache.spark.api.java.JavaSparkContext
import org.apache.hudi.utilities.streamer.HoodieStreamer
import org.apache.hudi.utilities.streamer.SchedulerConfGenerator
import org.apache.hudi.utilities.UtilHelpers
import com.beust.jcommander.JCommander
import com.beust.jcommander.Parameter

object GlueApp {

  def main(sysArgs: Array[String]) {
    val args = GlueArgParser.getResolvedOptions(sysArgs, Seq("JOB_NAME").toArray)

    val BUCKET = "XX"

    var config = Array(
      "--source-class", "org.apache.hudi.utilities.sources.ParquetDFSSource",
      "--source-ordering-field", "replicadmstimestamp",
      s"--target-base-path", s"s3://$BUCKET/testcases/",
      "--target-table", "invoice",
      "--table-type", "COPY_ON_WRITE",
      "--sync-tool-classes", "io.onetable.hudi.sync.OneTableSyncTool",
      "--enable-sync",
      "--hoodie-conf", "hoodie.datasource.write.recordkey.field=invoiceid",
      "--hoodie-conf", "hoodie.datasource.write.partitionpath.field=destinationstate",
      s"--hoodie-conf", s"hoodie.streamer.source.dfs.root=s3://$BUCKET/test/",
      "--hoodie-conf", "hoodie.datasource.write.precombine.field=replicadmstimestamp",
      "--hoodie-conf", "hoodie.onetable.formats.to.sync=ICEBERG",
      "--hoodie-conf", "hoodie.onetable.target.metadata.retention.hr=168"
    )

    val cfg = HoodieStreamer.getConfig(config)
    val additionalSparkConfigs = SchedulerConfGenerator.getSparkSchedulingConfigs(cfg)
    val jssc = UtilHelpers.buildSparkContext("delta-streamer-test", "jes", additionalSparkConfigs)
    val spark = jssc.sc

    val glueContext: GlueContext = new GlueContext(spark)
    Job.init(args("JOB_NAME"), glueContext, args.asJava)

    try {
      new HoodieStreamer(cfg, jssc).sync()
    } finally {
      jssc.stop()
    }

    Job.commit()
  }
}

```
# Add jar to jar path 
![Screenshot 2024-03-09 at 5 40 27 PM](https://github.com/soumilshah1995/onetable-deltastreamer-glue/assets/39345855/cec5c8ab-3828-4b8b-804f-07c03b2785ab)



