python的多线程一致饱受诟病（至少3.13之前）是因为GIL这个概念，即全局解释器锁，她确保同意时刻只有一个线程执行python字节码文件，其中一个原因就是因为python垃圾回收用的是计数器，你可以理解为修改计数器的方法外部套了个java的synchronized, 
这样保证了计数器天然安全，但是每个变量都有计数器，所以几乎一直是用一种单线程再跑多线程的代码。
一般也会把任务分成IO和CPU密集，本质上就是IO速度慢，这时候CPU执行完代码没事做，于是执行其他的代码，这时候A线程在等IO，B线程可以做点时候，GIL因为A暂停了于是在关注B，看似多线程实际上是因为等待。
CPU密集因为CPU一直在用，GIL会一直专注于活跃的那个线程 ，所以就变成了单线程

但是也是有办法绕开GIL，比如写一个consumer保存成一个py完后启动多次，或者使用multiprocessing，但是multiprocessing底层实现跟多开py脚本区别不大。。

asyncio 可以做到IO密集时候让CPU去做别的事情 比如DB读取
async def fetch_all_users(db_path):
    """查询所有用户"""
    async with aiosqlite.connect(db_path) as db:
        async with db.execute('SELECT id, name, age FROM users') as cursor:
            rows = await cursor.fetchall()
            return [{"id": r[0], "name": r[1], "age": r[2]} for r in rows]
