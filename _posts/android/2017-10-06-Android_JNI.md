---
layout: default
title: "Android_JNI"

category: android
istop: "true"
---

>     {{ page.date | date: "%Y-%m-%d," }} {{ page.content | number_of_words | append: "chars"}}
>     {{ page.tags }}

# Android_JNI

### libnativehelper/include/nativehelper/JniInvocation.hJniInvocation.h
导出了 libart的三个接口
{% highlight cpp %}
//非常重要的三个接口
  friend jint JNI_GetDefaultJavaVMInitArgs(void* vm_args);
  friend jint JNI_CreateJavaVM(JavaVM** p_vm, JNIEnv** p_env, void* vm_args);
  friend jint JNI_GetCreatedJavaVMs(JavaVM** vms, jsize size, jsize* vm_count);
{% endhighlight %}
其实现
### libnativehelper/JniInvocation.cpp
{% highlight cpp %}
...
//init初始化JniInvocation
bool JniInvocation::Init(const char* library) {
#ifdef __ANDROID__
  char buffer[PROP_VALUE_MAX];
#else
  char* buffer = NULL;
#endif
//查找libart.so
  library = GetLibrary(library, buffer);
  // Load with RTLD_NODELETE in order to ensure that libart.so is not unmapped when it is closed.
  // This is due to the fact that it is possible that some threads might have yet to finish
  // exiting even after JNI_DeleteJavaVM returns, which can lead to segfaults if the library is
  // unloaded.
  const int kDlopenFlags = RTLD_NOW | RTLD_NODELETE;
//open
  handle_ = dlopen(library, kDlopenFlags);
...
//找符号，赋给对应指针变量
  if (!FindSymbol(reinterpret_cast<void**>(&JNI_GetDefaultJavaVMInitArgs_),
                  "JNI_GetDefaultJavaVMInitArgs")) {
    return false;
  }
  if (!FindSymbol(reinterpret_cast<void**>(&JNI_CreateJavaVM_),
                  "JNI_CreateJavaVM")) {
    return false;
  }
  if (!FindSymbol(reinterpret_cast<void**>(&JNI_GetCreatedJavaVMs_),
                  "JNI_GetCreatedJavaVMs")) {
    return false;
  }
  return true;
}
...
//外部访问接口
extern "C" jint JNI_GetDefaultJavaVMInitArgs(void* vm_args) {
  return JniInvocation::GetJniInvocation().JNI_GetDefaultJavaVMInitArgs(vm_args);
}

extern "C" jint JNI_CreateJavaVM(JavaVM** p_vm, JNIEnv** p_env, void* vm_args) {
  return JniInvocation::GetJniInvocation().JNI_CreateJavaVM(p_vm, p_env, vm_args);
}

extern "C" jint JNI_GetCreatedJavaVMs(JavaVM** vms, jsize size, jsize* vm_count) {
  return JniInvocation::GetJniInvocation().JNI_GetCreatedJavaVMs(vms, size, vm_count);
}
{% endhighlight %}

### /frameworks/base/include/android_runtime/AndroidRuntime.h
### app_main.cpp
{% highlight cpp %}

{% endhighlight %}

### ZygoteInit

* zygote启动侦听socket端口，无限循环。采用poll机制，其接口定义在./libcore/luni/src/main/java/android/system/Os.java文件中，这个文件提供了访问linux系统调用的很多接口。
handler实现的底层用的也是epoll机制。 
* socket先建立链接`ZygoteConnection newPeer = acceptCommandPeer(abiList);`
* 然后读取数据并执行`boolean done = peers.get(i).runOnce();`
{% highlight java %}
    private static void runSelectLoop(String abiList) throws MethodAndArgsCaller {
        ArrayList<FileDescriptor> fds = new ArrayList<FileDescriptor>();
        ArrayList<ZygoteConnection> peers = new ArrayList<ZygoteConnection>();

        fds.add(sServerSocket.getFileDescriptor());
        peers.add(null);
       
        while (true) {
            StructPollfd[] pollFds = new StructPollfd[fds.size()];
            for (int i = 0; i < pollFds.length; ++i) {
                pollFds[i] = new StructPollfd();
                pollFds[i].fd = fds.get(i);
                pollFds[i].events = (short) POLLIN;
            }
            try {
                Os.poll(pollFds, -1);
            } catch (ErrnoException ex) {
                throw new RuntimeException("poll failed", ex);
            }
            for (int i = pollFds.length - 1; i >= 0; --i) {
                if ((pollFds[i].revents & POLLIN) == 0) {
                    continue;
                }
                //建立连接，新建一个socket，并放入fds中启动侦听
                if (i == 0) {
                    ZygoteConnection newPeer = acceptCommandPeer(abiList);
                    peers.add(newPeer);
                    fds.add(newPeer.getFileDesciptor());
                } else {
                    //执行创建子进程流程runOnce
                    boolean done = peers.get(i).runOnce();
                    if (done) {
                        peers.remove(i);
                        fds.remove(i);
                    }
                }
            }
        }
    }
{% endhighlight %}
