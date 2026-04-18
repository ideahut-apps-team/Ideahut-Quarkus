[__Ideahut Quarkus__](./index.md) <img height="20" src="./assets/ideahut.png" alt=""> <img height="20" src="./assets/quarkus.png" alt="">

# Task

Task Executor dan proses asinkronus.

``` java
public interface TaskHandler {

	// submit map callables, dan result diambil berdasarkan key
	Map<String, TaskResult> submit(Map<String, Callable<?>> callables, TimeValue timeout);
	Map<String, TaskResult> submit(Map<String, Callable<?>> callables);	
	
	// submit list callables, dan result diambil berdasarkan index
	List<TaskResult> submit(List<Callable<?>> callables, TimeValue timeout);
	List<TaskResult> submit(List<Callable<?>> callables);
	
	// submit single callable
	<T> Future<T> submit(Callable<T> callable);
	
	// execute multi tasks
	void execute(Collection<Runnable> tasks);

	// execute single task
	void execute(Runnable task);
}
```

## Bean

``` java
@Singleton
TaskHandler taskHandler(
	AppProperties appProperties		
) {
	TaskProperties properties = TaskDefinition.convert(appProperties.task().orElseThrow());
	return new TaskHandlerImpl()
	.setTaskProperties(properties);
}
```

### Properties
``` java
public class TaskProperties {
	private Boolean allowCoreThreadTimeOut;
	private TimeValue awaitTerminationTime;	
	private Integer corePoolSize;
	private Boolean daemon;
	private TimeValue keepAliveTime;	
	private Integer maxPoolSize;	
	private Integer queueCapacity;	
	private String threadNamePrefix;	
	private Integer threadPriority;	
	private Boolean waitForJobsToCompleteOnShutdown;
	private Boolean disableVirtualThread;
	private Boolean prestartAllCoreThreads;
}
```
- `allowCoreThreadTimeOut`: Ijinkan timeout thread saat eksekusi task (false / true).
- `awaitTerminationTime`: Waktu tunggu pada saat _shutdown_.
- `corePoolSize`: Minimum thread yang aktif di pool (meskipun tidak ada task yang dikerjakan).
- `daemon`: Task dikerjakan di latar atau tidak.
- `keepAliveTime`: Waktu tunggu thread selama eksekusi task.
- `maxPoolSize`: Maksimum thread yang aktif di pool.
- `queueCapacity`: Kapasitas antrian jika thread yang aktif di pool sudah penuh.
- `threadNamePrefix`: Prefix nama thread.
- `threadPriority`: Priotitas thread.
- `waitForJobsToCompleteOnShutdown`: Tunggu semua task selesai dieksekusi pada saat _shutdown_.
- `disableVirtualThread`: Tidak menggunakan Virtual Thread meskipun disupport oleh JDK yang dipakai.
- `prestartAllCoreThreads`: Start sejumlah thread sesuai dengan `corePoolSize`.

##

[__Ideahut Quarkus__](./index.md) <img height="20" src="./assets/ideahut.png" alt=""> <img height="20" src="./assets/quarkus.png" alt="">
