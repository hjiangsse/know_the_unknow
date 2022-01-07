CREATE DATABASE test_db ON CLUSTER 'clickhouse_cluster'

CREATE TABLE test_db.events ON CLUSTER 'clickhouse_cluster' (
    time DateTime,
    uid  Int64,
    type LowCardinality(String)
)
ENGINE = ReplicatedMergeTree('/clickhouse/tables/{cluster}/{shard}/table', '{replica}')
PARTITION BY toDate(time)
ORDER BY (uid);

CREATE TABLE test_db.events_distr ON CLUSTER 'clickhouse_cluster' AS test_db.events
ENGINE = Distributed('clickhouse_cluster', test_db, events, uid);

INSERT INTO test_db.events_distr VALUES ('2020-01-01 10:00:00', 100, 'view'),('2020-01-01 10:05:00', 101, 'view'),('2020-01-01 11:00:00', 100, 'contact'),('2020-01-01 12:10:00', 101, 'view'),('2020-01-02 08:10:00', 100, 'view'),('2020-01-03 13:00:00', 103, 'view'),('2020-01-03 14:00:00', 103, 'view'),('2020-01-03 15:00:00', 103, 'view'),('2020-01-03 16:00:00', 103, 'view'),('2020-01-03 17:00:00', 103, 'view'),('2020-01-03 18:00:00', 103, 'view');

sudo docker images | grep clickhouse | tr -s " " | cut -d " " -f 3 | xargs sudo docker image rm

非线性动量-scale_std$adjust_ll-roa_ttm_ind-scale_max$nan.h5
CREATE TABLE 非线性动量-scale_std$adjust_ll-roa_ttm_ind-scale_max$nan.h5 (x String) ENGINE = Memory AS SELECT 1;

Line #    Mem usage    Increment  Occurrences   Line Contents
=============================================================
    17  118.039 MiB  118.039 MiB           1   @profile
    18                                         def calculate_z_serial_purepython(maxiter, zs, cs):
    19                                             """Calculate output list using Julia update rule"""
    20  125.672 MiB    7.633 MiB           1       output = [0] * len(zs)
    21  128.777 MiB -4203.633 MiB     1000001       for i in range(len(zs)):
    22  128.777 MiB -4203.633 MiB     1000000           n = 0
    23  128.777 MiB -4203.867 MiB     1000000           z = zs[i]
    24  128.777 MiB -4203.625 MiB     1000000           c = cs[i]
    25  128.777 MiB -24020.664 MiB    34219980           while abs(z) < 2 and n < maxiter:
    26  128.777 MiB -22656.883 MiB    33219980               z = z * z + c
    27  128.777 MiB -19805.125 MiB    33219980               n += 1
    28  128.777 MiB -4804.148 MiB     1000000           output[i] = n
    29  128.777 MiB    0.000 MiB           1       return output


Filename: julia.py

Line #    Mem usage    Increment  Occurrences   Line Contents
=============================================================
    31   40.383 MiB   40.383 MiB           1   @profile
    32                                         def calc_pure_python(desired_width, max_iterations):
    33                                             """Create a list of complex coordinates(zs) and complex parameters(cs), build julia set, and display"""
    34   40.383 MiB    0.000 MiB           1       x_step = (float(x2 - x1) / float(desired_width))
    35   40.383 MiB    0.000 MiB           1       y_step = (float(y2 - y1) / float(desired_width))
    36   40.383 MiB    0.000 MiB           1       x = []
    37   40.383 MiB    0.000 MiB           1       y = []
    38
    39   40.383 MiB    0.000 MiB           1       ycoord = y1
    40   40.434 MiB    0.000 MiB        1001       while ycoord < y2:
    41   40.434 MiB    0.051 MiB        1000           y.append(ycoord)
    42   40.434 MiB    0.000 MiB        1000           ycoord += y_step
    43
    44   40.434 MiB    0.000 MiB           1       xcoord = x1
    45   40.695 MiB    0.086 MiB        1001       while xcoord < x2:
    46   40.695 MiB    0.176 MiB        1000           x.append(xcoord)
    47   40.695 MiB    0.000 MiB        1000           xcoord += x_step
    48
    49   40.695 MiB    0.000 MiB           1       zs = []
    50   40.695 MiB    0.000 MiB           1       cs = []
    51  118.035 MiB   -6.523 MiB        1001       for ycoord in y:
    52  118.035 MiB -6442.801 MiB     1001000           for xcoord in x:
    53  118.035 MiB -6472.348 MiB     1000000               zs.append(complex(xcoord, ycoord))
    54  118.035 MiB -10874.035 MiB     1000000               cs.append(complex(c_real, c_imag))
    55
    56  118.039 MiB    0.004 MiB           1       print("length of x: ", len(x))
    57  118.039 MiB    0.000 MiB           1       print("total elements: ", len(zs))
    58  118.039 MiB    0.000 MiB           1       start_time = time.time()
    59  128.781 MiB  128.781 MiB           1       output = calculate_z_serial_purepython(max_iterations, zs, cs)
    60  128.781 MiB    0.000 MiB           1       end_time = time.time()
    61  128.781 MiB    0.000 MiB           1       secs = end_time - start_time
    62  128.785 MiB    0.004 MiB           1       print("calculate_z_serial_purepython" + " took", secs, " secs")
