---
title: python多线程的使用
date: 2020-04-30 13:57:18
tags:
- python
- 多线程
categories: python
---

## 多线程适合使用的情景

类似于爬虫这种，需要等待请求的返回的情况，如果是同时读入多个文件并行处理的情形，还是使用multiprocessing模块比较合适

## 两种使用多进程的方式

[Python 线程池原理及实现](https://www.jianshu.com/p/afd9b3deb027) 说的比较详细，有几种线程池的模型，可以看看

### 直接用多线程

参考：

[Python多线程threading—图片下载](https://zhuanlan.zhihu.com/p/33521076)

本质上就是开了多个线程，同时运行。



主要步骤：

1. 开启线程（创建，`start`，存到一个thread_list中）
2. 等待所有线程都完成，程序结束（`t.join()`）



### 和queue搭配

参考

[Python 线程池原理及实现](https://www.jianshu.com/p/afd9b3deb027) 说的比较详细

[Python Multithreading and Multiprocessing Tutorial](https://www.toptal.com/python/beginners-guide-to-concurrency-and-parallelism-in-python)

主要的步骤有：

1. 创建Queue
2. 把相关的数据（比如待下载的url，函数参数等）`put`进Queue
3. 创建线程，start，每个线程都从queue中获取get到所需的变量（如上述的url，函数参数），执行（对应`run`），最后结束`queue.task_done()`
4. 当整个Queue都被运行之后，程序结束（即`Queue.join()`）



## 不合适的例子：用多线程分批从hdfs或者本地读入protobuf文件，然后根据条件进行筛选

[Python同时读取多个文件](https://zhuanlan.zhihu.com/p/131332807)

> 总结：
>
> - GIL作用：限制多线程同时执行，保证同一时刻只有一个线程执行。
> - 文件读写是会阻塞事件循环。

由于GIL锁的存在，即便是开启了多个线程，也仅仅在从hdfs get文件到本地时能够加快处理的速度。

实际上，由于一个时刻只能有一个文件被处理，因此在后续的读入protobuf文件以及根据gid进行筛选的过程中，各个线程是在争抢GIL锁的，并不合适

```python
class ThreadWorker(Thread):
    def __init__(self, queue, func, args, retry=1):
        super(ThreadWorker, self).__init__()
        self.queue = queue
        self.func = func
        self.retry = retry
        self.args = args

    def run(self):
        while True:
            if not self.queue.empty():
                retry_count = 0
                input_hour_dir = self.queue.get()
                while retry_count < self.retry:
                    try:
                        retry_count += 1
                        self.func(input_hour_dir, self.args)
                        break
                    except Exception as e:
                        print('Failed with %s' % e)
                if retry_count >= self.retry:
                    print('Failed after %d attempts' % retry_count)
                self.queue.task_done()
                print('Finished %s' % input_hour_dir)


def run(input_hour_dir, args):
    global all_gids
    valid_instances = 0
    all_instances = 0
    sizeof_size_t = 8

    if args.hdfs:
        [date, hour] = input_hour_dir.split('/')[-2:]
        output_name = 'valid_mp_' + '_'.join([date, hour]) + '.pb'
        hr = input_hour_dir + '/*.pb.snappy'
        tmp_file = os.path.join(args.output_dir, 'tmp_cascade_{}_{}'.format(date, hour))
        print('Hour file:%s, tmp_file:%s, output file: %s' % (hr, tmp_file, output_name))

        count_cmd = "hdfs dfs -ls {} | wc -l".format(hr)
        count_file_proc = Popen(count_cmd, shell=True, stderr=STDOUT, stdout=PIPE)
        count_file_proc.wait()
        if count_file_proc.returncode != 0:
            print('Failed to get number of files')
            return 1
        num_pb_snappy = int(count_file_proc.communicate()[0])
        print('Got %d .pb.snappy in %s' % (num_pb_snappy, hr))

        if not os.path.exists(tmp_file):
            if num_pb_snappy == 1:
                cmd = "hdfs dfs -get {} {}".format(hr, tmp_file)
            elif num_pb_snappy > 1:
                cmd = "hdfs dfs -getmerge {} {}".format(hr, tmp_file)
            else:
                print('Invalid hour file dir %s' % hr)
                return 1

            child_prc = Popen(cmd, stdout=PIPE, shell=True, stderr=STDOUT)
            child_prc.wait()
            if child_prc.returncode != 0:
                print('Fail to run cmd %s' % cmd)
                return 1
        else:
            print('Skip downloading file %s from hdfs' % tmp_file)
        decompressed_tmp = tmp_file + '.pb'
        if not os.path.exists(decompressed_tmp):
            cmd = 'python -m snappy -d {} {}'.format(tmp_file, decompressed_tmp)
            child_prc = Popen(cmd, shell=True, stderr=STDOUT, stdout=PIPE)
            child_prc.wait()
            
				# 从这一步开始，后续的步骤都会争抢GIL锁
        readfile = open(decompressed_tmp, 'r')
    else:
        output_name = 'valid_mp' + input_hour_dir.strip().split('.')[-2] + '.pb'
        print('Input hour file %s, output file name:%s' % (input_hour_dir, output_name))
        
        # 从这一步开始，后续的步骤都会争抢GIL锁
        readfile = open(input_hour_dir, 'r')
    instance = proto_parser_pb2.Instance()
    with open(os.path.join(args.output_dir, output_name), 'w') as writefile:
        while True:
            try:
								...
                # 有效小程序的样本
                if str(instance.line_id.gid) in all_gids: # 全局变量同样会被争抢
                    raw_proto.extend([size_binary[::-1], proto])
                    writefile.write(''.join(raw_proto))
                    valid_instances += 1

                if all_instances % 1000000 == 0:
                    print('Read {} instances in {}/{}'.format(all_instances, date, hr))
            except Exception as e:
                print(e)
                print('Got {} mp instances of {}/{} in {} '.format(valid_instances,
                                                                   date,
                                                                   hour,
                                                                   all_instances))
                break
                # print 'get [{}] instances success'.format(args.count)


def main(args):
    if args.hdfs:
        cmd = "hdfs dfs -ls {}".format(args.source_dir) + "| awk '{print $8}'"
        hour_list, _ = Popen(cmd, stdout=PIPE, shell=True, stderr=STDOUT).communicate()
        hour_list = hour_list.strip().split('\n')

        # cpus = multiprocessing.cpu_count()
        # cores = min(n_hours, cpus)
        #
        # pool = Pool(3)

    else:
        hour_list = os.listdir(args.source_dir)
        hour_list = [os.path.join(args.source_dir, hr) for hr in hour_list]

    n_hr_files = len(hour_list)
    print('Got %d files' % n_hr_files)

    global all_gids
    with open(args.gid_csv, 'r') as f:
        reader = csv.reader(f)
        next(reader)
        all_gids = set(x[0] for x in reader)

    queue = Queue()
    for hr in hour_list:
        queue.put(hr)

    num_threads = min(args.num_threads, n_hr_files)
    for _ in range(num_threads):
        thread = ThreadWorker(queue, run, args)
        thread.start()
    queue.join()

    print('Finished')


if __name__ == '__main__':
    parser = argparse.ArgumentParser()
    parser.add_argument('-dump_hdfs', dest='hdfs', action='store_true', help='Dump from hdfs')
    parser.add_argument('-source_dir', type=str, required=True, help='Source file dir.Can be local or hdfs')
    parser.add_argument('-output_dir', type=str, default='/home/tangweitao.walter/Download/test_multi_proc_dump_filter',
                        help='Output file location')
    parser.add_argument('-has_sort_id', type=int, default=1, help='has_sort_id')
    parser.add_argument('-kafka_dump', type=int, default=1, help='kafka_dump')
    parser.add_argument('-gid_csv', required=False, type=str, default="", help="gid_csv")
    parser.add_argument('-num_threads', required=False, type=int, default=3, help="number_of_threads")

    args = parser.parse_args()

    main(args)

```



 