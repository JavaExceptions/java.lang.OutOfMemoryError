java.lang.OutOfMemoryError
==========================

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

So you could go ahead and start taking thread dumps to analyze the situation. This would take days. Or you might try out 
Plumbr to find out what is causing this problem and how to cure it in just minutes.

