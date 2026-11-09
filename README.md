from pyspark.sql import SparkSession
from Tools.raw_vault_tests.engine import (
    find_source_business_keys_missing_from_hub,
)

spark = SparkSession.builder.appName("test_bk_source").getOrCreate()

# Source : trois Business Keys
source = spark.createDataFrame(
    [("BK001",), ("BK002",), ("BK003",)],
    ["bk_source"],
)

# HUB : BK003 volontairement absente
hub = spark.createDataFrame(
    [("BK001",), ("BK002",)],
    ["bk_hub"],
)

result = find_source_business_keys_missing_from_hub(
    source, hub, "bk_source", "bk_hub"
)

result.show(truncate=False)

assert result.count() == 1
assert result.first()["bk_source"] == "BK003"

print("TEST OK : BK003 détectée comme absente du HUB")

spark.stop()
