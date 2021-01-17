Bu yazıda .NET uygulamalarımızı monitör ederken, memory ve performans problemeleriyle karşılaştığımızda kullanabileceğimiz diagnostics araçlarını inceleyeceğiz. Daha öncesinde yine bu kapsamda <a href="https://ilkayilknur.com/net-core-uygulamalarini-dotnet-counters-ile-monitor-etmek-ve-custom-eventcounter-tanimlama" target="_blank">dotnet-counters</a> isimli aracı blogda yazmıştım. Dilerseniz o yazıyı da okuyabilirsiniz.

### dotnet-trace
`dotnet-trace` CLI toolu ile çalışan .NET uygulamaların tracelerini toplayabiliyoruz. Bu tool cross-platform olarak çalışabildiği için ve <a href="https://docs.microsoft.com/en-us/dotnet/core/diagnostics/eventpipe" target="_blank">`EventPipe`</a>'ları kullandığı için native profilera ihtiyaç duyulmuyor. 

Yüklemek için
`dotnet tool install --global dotnet-trace`

Toolu yükledikten sonra ilk yapmamız gereken şey tracelerini toplayacağımız .NET processini bulmak. Bunun için `dotnet-trace ps` komutunu kullanabiliriz. 

```bash
PS C:\Users\ilkay\source\repos\WebApplication27\WebApplication27\bin\Release\net5.0> dotnet trace ps
      6076 dotnet     C:\Program Files\dotnet\dotnet.exe
     27928 dotnet     C:\Program Files\dotnet\dotnet.exe
```

Process IDsini bulduktan sonra basitçe `dotnet-trace collect -p <pID>` komutunu çalıştırarak traceleri toplayabiliriz.

```bash
No profile or providers specified, defaulting to trace profile 'cpu-sampling'

Provider Name                           Keywords            Level               Enabled By
Microsoft-DotNETCore-SampleProfiler     0x0000F00000000000  Informational(4)    --profile
Microsoft-Windows-DotNETRuntime         0x00000014C14FCCBD  Informational(4)    --profile

Process        : C:\Program Files\dotnet\dotnet.exe
Output File    : C:\Users\ilkay\source\repos\WebApplication27\WebApplication27\bin\Release\net5.0\trace.nettrace

[00:00:00:04]   Recording trace 161.435  (KB)
Stopping the trace. This may take up to minutes depending on the application being traced.
```

Bu komutu çalıştırdıktan sonra traceler siz dur diyene kadar toplanacak ve dosyaya yazılacak. Bu nedenle bir noktada trace toplama işlemini sonlandırmak için `Enter` veya `Ctrl+C`ye basmanız gerekiyor.

`dotnet-trace` komutu default olarak cpu-sampling profile ile çalışmakta. Eğer GC profiling yapmak isterseniz size uygun olan profile'ı `dotnet-trace list-profiles` komutuyla bulup komutu çalıştırırken parametre geçebilirsiniz. `dotnet-trace` komutu default olarak .nettrace uzantılı dosyalar üretmekte. Bu dosyaları Visual Studio veya <a href="https://github.com/microsoft/perfview" target="_blank">PerfView</a> gibi araçlarla açıp analiz edebilirsiniz. Linux tarafındaki traceleri Windows'a aktarıp incelemek tabi ki bir opsiyon. Buna alternatif olarak komutu çağırırken `--format speedscope` belirtirseniz çıktı olarak üretilen trace dosyasını <a href="https://www.speedscope.app/" target="_blank">SpeedScope</a> üzerinden inceleyebilirsiniz. Varolan trace dosyalarını da başka formata çevirmek isterseniz yine `dotnet-trace convert` komutunu kullanmak mümkün.

.NET 5.0 ile beraber uygulamaların ayağa kalmasıyla beraber trace toplamak mümkün olmakta. Bu da bize uygulamaların startup zamanlarının monitör edebilmemize ve sorunları bulmamıza olanak sağlamakta. Bunun için `dotnet-trace collect -- <command>` şeklinde komutu çalıştırabiliriz. Bu komutu çalıştırdığımızda belirttiğimiz komut çalıştırılıp sonrasında traceler toplanmaya başlanır. Trace toplamayı durdurduğumuzda uygulama da kapatılır. 

`dotnet-trace collect -- WebApplication27.exe`

`dotnet-trace collect -- dotnet .\WebApplication27.dll`

`dotnet-trace` toolu ile ilgili daha detaylı bilgi almak için <a href="https://docs.microsoft.com/en-us/dotnet/core/diagnostics/dotnet-trace" target="_blank">buraya</a> bakabilirsiniz. 

### dotnet-dump

`dotnet-dump` CLI toolu bizim cross-platform olarak managed dump almamızı ve analiz etmemizi sağlayan bir tool. 

Yüklemek için 

`dotnet tool install --global dotnet-dump`

`dotnet-dump` toolunu kullanmak için `dotnet-trace` toolunda olduğu gibi dumpını alacağımız processin IDsini veya adını bulmamız gerekiyor. Sonrasında komutu çağırırken `-p` veya `-n` ile ilgili parametreyi geçebiliriz.

Dump almak için `dotnet-dump` tooluna `collect` komutunu geçerek ilgili parametreleri de belirtmemiz gerekiyor. 

Örneğin, process IDsi ile dump almak istersek.

`dotnet-dump collect -p 10956`

Aynı zamanda dump alırken `--type` parametresinde hangi tipte(Mini, Heap, Full) bir dump almak istediğimizi de belirtebiliriz. Eğer belirtmezsek full dump alınmakta.

`dotnet-dump collect -p 10956 --type Mini`

Dump aldıktan sonra alınan dumpı analiz etmek için de `analyze` komutunu çalıştırıp <a href="https://docs.microsoft.com/en-us/dotnet/core/diagnostics/dotnet-dump#analyze-sos-commands" target="_blank">SOS komutlarıyla</a> analiz edebiliriz.

`dotnet-dump analyze <DumpDosyasi>`

`Analyze` komutunu çağırdıktan sonra tool bizden analiz için kullanabileceğimiz komutları bekliyor. Kullanabileceğimiz komutlara `help` yazarak da ulaşmamız mümkün. 

```bash
PS C:\Users\ilkay\source\repos\WebApplication27\WebApplication27\bin\Release\net5.0> dotnet-dump analyze .\dump_20210117_173351.dmp
Loading core dump: .\dump_20210117_173351.dmp ...
Ready to process analysis commands. Type 'help' to list available commands or 'help [command]' to get detailed help on a command.
Type 'quit' or 'exit' to exit the session.
```
  
```bash
> verifyheap
No heap corruption detected.
> threadpool
CPU utilization: 2 %%
Worker Thread: Total: 1 Running: 0 Idle: 1 MaxLimit: 32767 MinLimit: 12
Work Request in Queue: 0
--------------------------------------
Number of Timers: 1
--------------------------------------
Completion Port Thread:Total: 2 Free: 2 MaxFree: 24 CurrentLimit: 2 MaxLimit: 1000 MinLimit: 12
```

`dotnet-dump` ile aldığımız dumpların bir kısıtı da örneğin Linux tarafında aldığımız dumpları Windows tarafında inceleyemememiz. Bu durumun tam tersi de geçerli. Bu da bizi bir sonra inceleyeceğimiz komuta götürecek.

`dotnet-dump` toolu ile ilgili daha detaylı bilgi için <a href="https://docs.microsoft.com/en-us/dotnet/core/diagnostics/dotnet-dump" target="_blank">burayı</a> inceleyebilirsiniz.

### dotnet-gcdump

`dotnet-gcdump` CLI toolu ile çalışan uygulamaların gcdumplarını alabiliyoruz. Bu tool ile gcdump alınırken Gen2 GC tetikleniyor ve EventPipe üzerinde gelen eventler kullanılıyor. Bu nedenle `dotnet-dump` komutundaki limitlemeler burada yok ve daha lightweight bir çözüm. Cross-platform şeklinde inceleme yapabiliyoruz. Tool çalışması sırasında Gen2 GC tetiklediği için uygulamada Gen2'un fazla büyük olduğu senaryolarda uygulamada duraksamaların olması beklenen bir davranış. Bu nedenle performans kritik applerde bu komutun çalıştırılması problemler oluşturabilir.

Yüklemek için 

`dotnet tool install --global dotnet-gcdump`

Dump almak için 

`dotnet-gcdump collect -p <pID>`

Bu tooldan aldığımız gcdumpları Visual Studio veya <a href="https://github.com/microsoft/perfview" target="_blank">PerfView</a> kullanarak analiz edebiliriz.

![visual-studio-gc-dump-analyze](https://ilkayblog.blob.core.windows.net/uploads/2021/01/17/gcdump-analyze.png)

Bu yazıda basitçe uygulamalarımızı monitör ederken veya çeşitli sorunlarla karşılaştığımızda kullabileceğimiz toollardan bahsetmeye çalıştım. Tabi ki biz bu yazıda basitçe komutlardan ve ne işe yaradıklarından bahsettik. Production ortamlarında bu komutları ve araçları bilmenin yanı sıra bu araçların ürettiği çıktıları da analiz etmek oldukça önemli. İlerleyen yazılarda bu konulardan da bahsetmeyi planlıyorum. 

Bir sonraki yazıda görüşmek üzere,

Kaynaklar
* https://github.com/dotnet/diagnostics
* https://docs.microsoft.com/en-us/dotnet/core/diagnostics/dotnet-dump
* https://docs.microsoft.com/en-us/dotnet/core/diagnostics/dotnet-gcdump
* https://docs.microsoft.com/en-us/dotnet/core/diagnostics/dotnet-trace