# about python package:
## pack a python package:
    a good link page i know to pack a package:  https://packaging.python.org/tutorials/packaging-projects/
## install python package faster
    pip install --force-reinstall rdsp_client-0.0.4-py3-none-any.whl -i https://pypi.tuna.tsinghua.edu.cn/simple

# panda dataframe
## pandas data frame filter by index
a pandas dataframe *data*:
                         open     close      high       low     value    volume
2005-01-01 09:00:00  0.552438  0.280006  0.332489  0.864800  0.474178  0.827809
2005-01-01 09:01:00  0.338546  0.654993  0.659209  0.888915  0.480564  0.651820
2005-01-01 09:02:00  0.962025  0.463558  0.092408  0.296262  0.284668  0.448517
2005-01-01 09:03:00  0.874030  0.560000  0.261885  0.534373  0.747222  0.849829
2005-01-01 09:04:00  0.816664  0.541799  0.288249  0.005849  0.881951  0.320970
...                       ...       ...       ...       ...       ...       ...
2005-01-01 10:45:00  0.018078  0.270889  0.007631  0.856010  0.886102  0.501975
2005-01-01 10:46:00  0.299404  0.692678  0.894732  0.307676  0.606485  0.598515
2005-01-01 10:47:00  0.116287  0.597343  0.722067  0.811261  0.061974  0.193382
2005-01-01 10:48:00  0.434326  0.493112  0.003419  0.900076  0.049911  0.114951
2005-01-01 10:49:00  0.524980  0.527988  0.546563  0.498139  0.226002  0.306714

在上述dataframe中，我想过滤出2005-01-01 10:45:00之后的所有数据， how?
``` python
data_stamps = list(data.index.values)

last_row_stamp = '2005-01-01 10:45:00'
insert_stamps_filtered = filter(lambda x: x > last_row_stamp, data_stamps)
insert_stamps_list = list(insert_stamps_filtered)

filtered_data = data.filter(items = insert_stamps_list, axis=0)
```
