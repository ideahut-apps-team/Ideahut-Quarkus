[__Ideahut Quarkus__](./index.md) <img height="20" src="./assets/ideahut.png" alt=""> <img height="20" src="./assets/quarkus.png" alt="">

# Launcher

Class yang menjadi titik awal untuk menjalankan aplikasi.

``` java
@QuarkusMain
public class Application implements WebLauncher {
	public static class Package {
		private Package() {}
		public static final String LIBRARY		= FrameworkHelper.PACKAGE;
		public static final String APPLICATION	= "net.ideahut.quarkus.template";
	}
	
	private static boolean ready = false;
	private static void setReady(boolean b) { ready = b; }
	public static boolean isReady() { return ready; }
	
	public static void main(String... args) {
		WebLauncher.runApp(Application.class, args);
	}
	
	// Startup, wajib ada dengan parameter '@Observes StartupEvent event'
	@Override
	public void onStartup(@Observes StartupEvent event, Router router) {
		WebLauncher.super.onStartup(event, router);
	}
	
	// Shutdown, wajib ada dengan parameter '@Observes ShutdownEvent event'
	@Override
	public void onShutdown(@Observes ShutdownEvent event) {
		WebLauncher.super.onShutdown(event);
	}
	
	// mendapatkan definisi dari konfigurasi
	@Override
	public LauncherDefinition onDefinition() {
		return FrameworkHelper.getBean(AppProperties.class).launcher().orElse(null);
	}
	
	// inisialisasi dan konfigurasi bean telah selesai dan sukses
	@Override
	public void onReady() {
		setReady(true);
		NativeConfig.registerToNativeImageAgent();
	}
	
	// error launcher
	@Override
	public void onError(Throwable throwable) {
		log.error("Application", throwable);
		Quarkus.asyncExit();
		System.exit(0);
	}
	
	// log dari proses launcher
	@Override
	public void onLog(LauncherDefinition.Log.Type type, LauncherDefinition.Log.Level level, String message, Throwable throwable) {
		level = ObjectHelper.useOrDefault(level, () -> LauncherDefinition.Log.Level.DEBUG);
		log.atLevel(org.slf4j.event.Level.valueOf(level.name())).log(message, throwable);
	}
	
	// untuk menambah path dan handler-nya bisa dilakukan di method ini.
	// sebagai contoh di bawah, menambahkan handler untuk Web Admin
	@Override
	public void onRouter(Router router) {
		AppProperties appProperties = FrameworkHelper.getBean(AppProperties.class);
		AdminHandler adminHandler = FrameworkHelper.getBean(AdminHandler.class);
		ObjectHelper.runIf(
			null != adminHandler, 
			() -> {
				SecurityCredential adminCredential = FrameworkHelper.getBean(AppConstants.Bean.Credential.ADMIN, SecurityCredential.class);
				WebHelper.Router.admin(router, adminHandler, adminCredential, appProperties.publicBaseUrl().orElse(null));
			}
		);
		DataMapper dataMapper = FrameworkHelper.getBean(DataMapper.class);
		WebHelper.Router.root(router, appProperties.filter().orElse(null), dataMapper, () -> FrameworkHelper.randomAlphanumeric(10), Application::isReady);
	}
	
}
```

## Properties
- Class 'net.ideahut.quarkus.definition.LauncherDefinition'.
```md
launcher:
    # Tunggu semua proses inisialisasi & konfigurasi sampai selesai
	waitAllProcess: true
	
	# menjalankan fitur yang ada di 'net.ideahut.quarkus.init.InitHandler'
	init:
	    # meregister obyek-obyek yang digunakan ke dalam mapper
		mapper: true
		# eksekusi 'jakarta.validation.Validator'
		validation: true
		# inisialisasi endpoint (warmup)
		endpoint: true
	
	# daftar log yang diaktifkan
	log:
		initHandler: true
		beanInitialize: true
		beanConfigure: true
		beanShutdown: true
		version: true
		application: true
	
	# Konfigurasi bean
	configure:
	    # max thread untuk BeanConfigure yang terdeteksi secara otomatis dari quakus context (selain yang didefinisikan di beans)
		concurrency: 0
		# daftar bean yang dikonfigurasi duluan secara berurutan, karena ada kemungkinan dibutuhkan oleh bean-bean yang lain
		beans:
			- net.ideahut.quarkus.collector.RequestInfoCollector
            - net.ideahut.quarkus.entity.EntityTrxManager
            - net.ideahut.quarkus.sysparam.SysParamHandler
            - net.ideahut.quarkus.audit.AuditHandler
```

##

[__Ideahut Quarkus__](./index.md) <img height="20" src="./assets/ideahut.png" alt=""> <img height="20" src="./assets/quarkus.png" alt="">
