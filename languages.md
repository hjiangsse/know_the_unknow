# golang install from apt-get
sudo add-apt-repository ppa:ubuntu-lxc/lxd-stable
sudo apt-get update
sudo apt-get -y upgrade
sudo apt-get install golang

# c++ abseil lib from google
https://abseil.io/docs/cpp/quickstart-cmake

# when your program can not find the dynamic library(.so) do the fllowing:
## find library file location   
    sudo find / -name libclickhouse-cpp-lib.so 

    example:
    jingle@jingle-nas-220:~/hjiang$ sudo find / -name libclickhouse-cpp-lib.so
    /home/jingle/hjiang/clickhouse-cpp/build/clickhouse/libclickhouse-cpp-lib.so
    /usr/local/lib/libclickhouse-cpp-lib.so

## add lib path to LD_LIBRARY_PATH
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib
add this line to ~/.bashrc
then source ~/.bashrc

## then use the new lib
g++ --std=c++17 test_click.cpp -lclickhouse-cpp-lib

# c++ closure use point as parameter
``` c++
char *group_start =  pmmap_bak + timestamp_len;
for(group_iter it = groups.begin(); it != groups.end(); it++) {
    string select_query = gen_select_query(*it, database_name, table_name, start_date, end_date); //生成这个股票小组的查询请求
    int64_t group_len = (*it).size();                                                             //本小组含有的股票个数

    m_threads.push_back(std::thread(select_group_then_write_memory, clickhouse_addr, select_query, group_start, group_len, value_len, row_byte_size));
    group_start = group_start + group_len * value_len;                              //移动到下一个小组的起始位置
}

/*
*  查询symbol_group列，并将结果写入内存之中
*/ 
void select_group_then_write_memory(const string &addr, const string &query, char *shared, uint64_t group_len, uint64_t factor_val_size, uint64_t row_byte_size) {
    Client client(ClientOptions().SetHost(addr));
    //std::lock_guard<std::mutex> guard(mtx);

    char **pshared = &shared;
    client.Select(query, [pshared, group_len, factor_val_size, row_byte_size] (const Block& block) {
        char *pmm = *pshared;
        for(size_t i = 0; i < block.GetRowCount(); i++) {
            for(int j = 0; j < group_len; j++) {
                memcpy(pmm, (char*)&(block[j]->As<ColumnFloat64>()->At(i)), factor_val_size);
                pmm += factor_val_size;
            }
            pmm += (row_byte_size - group_len * factor_val_size);
        }
        *pshared = pmm;
    });
}
```


