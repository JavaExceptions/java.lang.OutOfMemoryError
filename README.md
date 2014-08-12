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
