java.lang.OutOfMemoryError
==========================

OutOfMemoryError (OOM) - É o tipo de erro relacionado à insuficiência de memória e ocorre devido à algum memory leak no código da aplicação. Para analisar a causa desse tipo de erro é interessante ter em mãos o estado (snapshot) do Heap da JVM no momento em que o erro ocorre. Para isso é necessário habilitar alguns parâmetros na JVM conforme abaixo:

Geração do Dump (formato hprof) do Heap via parâmetro da JVM.

Adicionar as seguintes propriedades no comando que inicia a JVM. Caso o comando seja parametrizado em um arquivo de configuração, basta usar uma variável JAVA_OPTS como no exemplo abaixo.

  # HeapDUMP e Core DUMP
  # registra a data e hora de início do processo
  DATE_START=`date  +%d-%m-%Y-%k%M`

  # Habilita a geração do DUMP de memória (HPROF)
  JAVA_OPTS="$JAVA_OPTS -XX:+HeapDumpOnOutOfMemoryError "
  JAVA_OPTS="$JAVA_OPTS -XX:HeapDumpPath=/var/log/jvm/heapdump_$DATE_START.hprof "

NOTA: o arquivo de dump no formato .hprof pode ser aberto com os utilizatários jVisualvm (fornecido pelo JDK Sun/Oracle Hotspot) ou Eclipse Memory Analizer (MAT). É necessário que a versão e arquitetura do JDK utilizado na análise dump seja idêntica à JVM utilizada pelo servidor de aplicação onde o erro ocorreu.

Gerando dumps da JVM em tempo de runtime manualmente:

  $ jmap -dump:live,format=b,file=jvm_heap.bin <PID>

O formato binário pode ser aberto com as ferramentas: jstack (JDK), jVisualVM (JDK), Eclipse MAT [ref:1,2] ou qualquer outro profiler Java que reconheça o formato HPROF.

  [1] http://help.eclipse.org/indigo/index.jsp?topic=%2Forg.eclipse.mat.ui.help%2Ftasks%2Facquiringheapdump.html
  [2] http://www.eclipse.org/mat/downloads.php

O formato texto pode ser aberto com as ferramentas: jVisualVM (JDK), IBM Thread Analizer ou qualquer outro profiler Java que reconheça dump de threads Java.

NOTAS:

    O arquivo de dump no formato hprof pode ser aberto com os utilizatários jVisualvm (fornecido pelo JDK) ou Eclipse Memory Analizer (MAT).  É necessário que a versão e arquitetura do JDK utilizado na análise dump seja idêntica à JVM utilizada pelo servidor de aplicação onde o erro ocorreu.
    Para que o comando gcore (invocado pela JVM após o evento de erro) funcione é necessário que o pacote gdb (A GNU source-level debugger for C, C++, Fortran and other languages) esteja devidamente instalado no Sistema Operacional.
    O procedimento acima causam o travamento das threads (equivalente ao efeito Stop The World do FullGC). Portanto utilize com cautela em ambiente de produção!
-------------------------------------------------------------
Resolution as Red Hat:

Decrease the JVM process size:
    Explicitly set the thread stack size. The reason for this is that the default value can be 1024kB and a typical thread stack size of 128K or 256K is usually adequate. See How do I set Java thread stack size?.
    Decrease the amount of memory reserved for the heap and/or perm gen space.
    Upgrade from JDK 1.5 to JDK 1.6 or OpenJDK 1.6 to take advantage of the more efficient memory mapping of jar files that significantly reduces JVM memory consumption. JDK 1.5 memory maps entire jar files, whereas JDK 1.6 and OpenJDK 1.6 only memory maps a central directory.
    Use a preferred or supported Windows service wrapper to ensure that options are being passed to the JVM. See What is the preferred method of running JBoss AS as a Windows Service?.

    Increase the amount of available and/or contiguous address space:
    For 32-bit Windows, set the 3GB switch.
        Not supported as of JDK 1.6.
        Can potentially cause "out of kernel address space" issues under heavy network/io load.
    Uninstall any non-essential software to avoid shared libraries with preferred memory loading locations like Windows DLLs from fragmenting address space.
    Move to 64-bit where there is no memory split and no 4GB limit.

    Increase OS limits on the number of threads:
    Set a higher ulimit for open files in /etc/security/limits.conf on Linux. For example:

        soft    nofile           1024
        hard    nofile           8192

    or to set both hard and soft to the same value:

        - nofile 2048

    Set higher ulimit for max user processes in /etc/security/limits.conf on Linux.
    For example:

        soft    nproc           2048
        hard    nproc           8192
        
    For RHEL 6, a modification to /etc/security/limits.d/90-nproc.conf is needed rather than /etc/security/limits.conf, refer article for more details.

    If the maximum number of threads will increase beyond the default allowed (32768) you will need to increase this limit in /etc/sysctl.conf. This value is controlled by pid_max, and to double this value the following line would be added:

        kernel.pid_max = 65536

    Decrease the number of threads.
    If you have applications that use JSF, try the following settings in their web.xml:

  <context-param>
    <param-name>facelets.REFRESH_PERIOD</param-name>
    <param-value>-1</param-value>
  </context-param>
  <context-param>
    <param-name>facelets.DEVELOPMENT</param-name>
    <param-value>false</param-value>
  </context-param>

    Avoid running multiple copies of the same application by ensuring it's fully stopped before restarting.
    
-------------------------------------------------------------
The following example is creating and starting new threads in a loop. When running the code, operating system limits 
are reached fast and “java.lang.OutOfMemoryError: Unable to create new native thread” message will be displayed.

while(true){
    new Thread(new Runnable(){
        public void run() {
            try {
                Thread.sleep(10000000);
            } catch(InterruptedException e) { }        
        }    
    }).start();
}

Solution for java.lang.OutOfMemoryError:

On occasions you can bypass the “Unable to create new native thread” issue by increasing the limits on the OS level. 
For example, if you have limited the number of processes that the JVM can spawn in user space you should check out and 
possibly increase the limit:

[root@dev ~]# ulimit -a
core file size          (blocks, -c) 0
--- cut for brevity ---
max user processes              (-u) 1800

More often than not, the limits on new native threads hit by the OutOfMemoryError indicate a programming error. 
When your application is spawning thousands of threads then chances are that something has gone terribly wrong – there are 
not many applications out there which would benefit from such a vast amount of threads.

-------------------- 
# http://stackoverflow.com/questions/65200/how-do-you-crash-a-jvm
# http://middlewaremagic.com/weblogic/?p=4482

These are just normal exceptions. To really crash a VM there are 3 ways:

Use JNI and crash in the native code.
If no security manager is installed you can use reflection to crash the VM. This is VM specific, but normally a VM stores a bunch of pointers to native resources in private fields (e.g. a pointer to the native thread object is stored in a long field in java.lang.Thread). Just change them via reflection and the VM will crash sooner or later.
All VMs have bugs, so you just have to trigger one.
For the last method I have a short example, which will crash a Sun Hotspot VM quiet nicely:

public class Crash {
    public static void main(String[] args) {
        Object[] o = null;

        while (true) {
            o = new Object[] {o};
        }
    }
}
This leads to a stack overflow in the GC so you will get no StackOverflowError but a real crash including a hs_err* file.
