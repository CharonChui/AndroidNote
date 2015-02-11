任务管理器(ActivityManager)
===

`Android`中**ActivityManager**类似于`Windows`下的任务管理器，能得到正在运行程序的内容等信息    

1. List<ActivityManager.RunningServiceInfo>  getRunningServices(int maxNum)     
    Return a list of the services that are currently running.
	这个maxNum是指返回的这个集合的最大值    
	可以利用`ActivityManager`去判断当前某个服务是否正在运行。

2. List<ActivityManager.RunningAppProcessInfo>  getRunningAppProcesses()     
    Returns a list of application processes that are running on the device.

3. List<ActivityManager.RecentTaskInfo>  getRecentTasks(int maxNum, int flags)     
    得到最近使用的程序，集合中第一个元素是刚才正在使用的

4. Debug.MemoryInfo[]  getProcessMemoryInfo(int[] pids)      
	Return information about the memory usage of one or more processes.
	可以通过某个进程的id得到进程的内存使用信息，然后通过这个内存信息能够得到每个程序使用的内存大小

	MemoryInfo中的方法     
    int getTotalPrivateDirty()            
	Return total private dirty memory usage in kB得到占用内存的大小，单位是kb    

	```java
    /**
     * 返回所有的进程列表信息
     * @param context
     * @return
     */
    public static List<TaskInfo> getTaskInfos(Context context){
        ActivityManager am = (ActivityManager) context.getSystemService(Context.ACTIVITY_SERVICE);
        List<RunningAppProcessInfo> appProcessInfos = am.getRunningAppProcesses();
        List<TaskInfo> taskInfos = new ArrayList<TaskInfo>();
        PackageManager pm = context.getPackageManager();
        for(RunningAppProcessInfo appProcessInfo : appProcessInfos){
            String packname = appProcessInfo.processName;
            TaskInfo taskInfo = new TaskInfo();
            taskInfo.setPackname(packname);
            
            MemoryInfo[] memoryInfos = am.getProcessMemoryInfo(new int[]{appProcessInfo.pid});
            long memsize = memoryInfos[0].getTotalPrivateDirty() * 1024;
            taskInfo.setMemsize(memsize);
            try {
                PackageInfo packInfo = pm.getPackageInfo(packname, 0);
                Drawable icon = packInfo.applicationInfo.loadIcon(pm);
                taskInfo.setIcon(icon);
                String name = packInfo.applicationInfo.loadLabel(pm).toString();
                taskInfo.setName(name);
                if(AppInfoProvider.filterApp(packInfo.applicationInfo)){
                    taskInfo.setUserTask(true);
                }else{
                    taskInfo.setUserTask(false);
                }
            } catch (NameNotFoundException e) {
                taskInfo.setIcon(context.getResources().getDrawable(R.drawable.ic_launcher));
                taskInfo.setName(packname);
                e.printStackTrace();
            } 
            taskInfos.add(taskInfo);
        }
        return taskInfos;
    }
    ```

5. 一键清理     
	杀死进程需要权限     
	```xml
	android.permission.KILL_BACKGROUND_PROCESSES
	```
	杀死进程就是使用ActivityManager的killBackgroundProcess方法
	```
	public void killBackgroundProcesses(String packageName)
	```

6. 获取内存可用大小
    ```java
	public class ProcessStatusUtils {
        /**
         * 获取有多少个程序正处于运行状态.
         * @param context
         * @return
         */
        public static int getProcessCount(Context context){
            ActivityManager am = (ActivityManager) context.getSystemService(Context.ACTIVITY_SERVICE);
            return am.getRunningAppProcesses().size();
        }
    
        /**
         * 获取手机里面可用的内存空间
         * @param context
         * @return long类型的byte的值
         */
        public static long getAvailRAM(Context context){
            ActivityManager am = (ActivityManager) context.getSystemService(Context.ACTIVITY_SERVICE);
            MemoryInfo outInfo = new MemoryInfo();
            am.getMemoryInfo(outInfo);
            return outInfo.availMem;
        }

        //获取手机的总内存，Android的Api中没有提供获取总内存的方法，在linux系统中我们要通过这个文件才能得到总内存
        public static long getTotalRAM(){
            try {
                File file = new File("/proc/meminfo");//Android系统这个文件的第一行就能得到总的内存大小
                FileInputStream fis = new FileInputStream(file);
                BufferedReader br = new BufferedReader(new InputStreamReader(fis));
                String str = br.readLine();
                //MemTotal:         513248 kB
                char[] chars = str.toCharArray();
                StringBuffer sb = new StringBuffer();
                for(char c : chars){
                    if(c>='0'&&c<='9'){
                        sb.append(c);
                    }
                }
                return Integer.parseInt(sb.toString())*1024;//得到的是多少kb,将kb转成b
            } catch (Exception e) {
                e.printStackTrace();
                return 0;
            }
        }
    } 
    //上面的方法都是得到的多少比特的大小，在使用中可以使用Formatter.formatFileSize(Context context, long b)将其自动转成K,M,G等
    ```
    
---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 