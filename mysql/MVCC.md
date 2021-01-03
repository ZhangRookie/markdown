#### MVCC

- 脏写：A事务修改了未提交的B事务修改的数据
- 赃读：A事务读到了未提交的B事务修改的数据
- 不可重复度：一个事务只能读到另一个已提交事务修改过的数据，说明发生了不可重复读
- 幻读：A事务根据条件查出一些数据，B事务向表中插入满足条件的数据，A事务按条件再次查询能够查出新数据，则发生幻读



__MVCC原理__ 

__版本链：__

- trx_id：每次一个事务对数据进行修改都会将事务id写到trx_id列
- roll_pointer：每次一个事务对数据进行修改，都会新增一条undo日志，记录修改前的信息，roll_pointer指向这个日志

经过多次修改，所有版本都会链接在一起，每条日志都存有对应的事务ID，形成一个版本链

__ReadView：__

READ_UNCOMMITED可以读未提交事务的数据，所以可以直接读最新数据，SERIALIZABLE使用锁来读取数据

对于READ_COMMITED和REPEATABLE_READ必须保证读到已提交事务修改记录，不能读到未提交事务数据

- m_ids：表示生成ReadView时活跃事务id列表
- min_trx_id：表示生成ReadView时最小的事务id，m_ids中的最小值
- max_trx_id：表示生成readView时系统下一个应该分配的事务id
- creator_trx_id：表示生成readView的事务id

__判断版本可见性依据：__

- 如果日志的trx_id等于creator_trx_id，则说明日志为当前事务修改的版本，可读
- 如果日志trx_id 小于min_trx_id，说明对应版本事务在当前事务之前且已提交，可读
- 如果日志trx_id大于或等于max_trx_id，说明对应版本事务在当前事务之后，不可读
- 如果日志trx_id在max_trx_id和min_trx_id之间，需要判断是否在m_ids中，如果在，说明在生成ReadView时该事务活跃，不可读，否则说明事务已提交，可读
- 如果不可见，就向下继续读，如果最后一个版本仍不可见，则说明当前记录对该事务不可见

READ_COMMITED 在每次读之前都会生成ReadView

REPEATABLE_READ在第一次读之前生成ReadView

