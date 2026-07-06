python的多线程一致饱受诟病（至少3.13之前）是因为GIL这个概念，即全局解释器锁，她确保同意时刻只有一个线程执行python字节码文件，其中一个原因就是因为python垃圾回收用的是计数器，你可以理解为修改计数器的方法外部套了个java的synchronized, 
这样保证了计数器天然安全，但是每个变量都有计数器，所以几乎一直是用一种单线程再跑多线程的代码。

这会分为2个不同场景：
1. 代码中涉及文件读写，网络访问等，这种时间更多消耗在传输，因为CPU是毫秒级别，而网络和磁盘是秒级别，这时候并没有变量的修改，CPU也是一种等待状态，此时在python使用多线程可以获取一定的效果。
2. 代码中涉及大量多重计算，比如变量的转换，修改，计算等，此时变量一直被修改和引用或者释放，GIL再每一次修改都会运行一次，此时多线程基本是一个接近于单线程的状态，可以考虑多进程，比如用kafka做消息队列，下游多个消费者处理。


但是也是有办法绕开GIL，比如写一个consumer保存成一个py完后启动多次，或者使用multiprocessing，但是multiprocessing底层实现跟多开py脚本区别不大。。

asyncio 可以做到IO密集时候让CPU去做别的事情 比如DB读取
async def fetch_all_users(db_path):
    """查询所有用户"""
    async with aiosqlite.connect(db_path) as db:
        async with db.execute('SELECT id, name, age FROM users') as cursor:
            rows = await cursor.fetchall()
            return [{"id": r[0], "name": r[1], "age": r[2]} for r in rows]
