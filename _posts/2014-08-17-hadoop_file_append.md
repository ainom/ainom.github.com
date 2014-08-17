---
layout: post
date: "2014-08-17 23:49:PM"
title: "Hadoop 文件追加"
description: "Hadoop 0.20之前的版本应该不支持文件追加功能，我用的是1.0版本的。要想使用文件追加写入功能，先配置hdfs-site.xml。"
category: hadoop
tags: [hadoop]
---
{% include JB/setup %}

Hadoop 0.20之前的版本应该不支持文件追加功能，我用的是1.0版本的。要想使用文件追加写入功能，先配置hdfs-site.xml,如下：
</br>     \<property\>
</br>        \<name\>dfs.support.append\</name\>
</br>        \<value\>true\</value\>
</br>     \</property\>
</br>dfs.support.append默认是关闭的。
</br>然后程序打开文件时用：
</br>FileSystem fs = FileSystem.get(URI.create(dst), conf);
</br>FSDataOutputStream out = fs.append(new Path(dst));
</br>在网上找了一天都没有找到解决方法，而且都说hadoop没有这个功能。最后没办法看了下异常，突然发现异常中用个dfs.support.append设置的提示，配置了下，居然成功了，特此分享。



</br>import org.apache.hadoop.conf.Configuration; 
</br>import org.apache.hadoop.fs.FileSystem; 
</br>import org.apache.hadoop.fs.Path; 
</br>import org.apache.hadoop.io.IOUtils; 

</br>import java.io.*; 
</br>import java.net.URI; 
</br>/** 
</br>* 
</br>* @description: 
</br>* @class: AppendContent.java 
</br>* @author <a href="mailto:javamickey@163.com">javamickey</a> 
</br>* @date 2014年8月13日 下午5:39:39 
</br>* @version 1.0 
</br>*/ 
</br>public class AppendContent { 
</br>public static void main(String[] args) { 
</br>	System.out.println("---start merge---"); 
</br>	String hdfs_path = "hdfs://0.0.0.0/tmp/logs/ttk/ttk_show/2014-08-11/ttk_show.log.2014-08-11-01.log";//文件路径 
</br>	Configuration conf = new Configuration(); 
</br>	conf.setBoolean("dfs.support.append", true); 
</br>	conf.setBoolean("dfs.support.broken.append", true); 
</br>	conf.setBoolean("dfs.client.block.write.replace-datanode-on-failure.policy.enable", false); 
</br>	conf.set("dfs.client.block.write.replace-datanode-on-failure.policy", "NEVER"); 
</br>	String inpath = "/home/append.txt"; 
</br>	FileSystem fs = null; 
</br>try { 
</br>	System.out.println("--------------------start-------------------"); 
</br>	fs = FileSystem.get(URI.create(hdfs_path), conf); 
</br>	//要追加的文件流，inpath为文件 
</br>	fs = FileSystem.get(URI.create(hdfs_path), conf); 
</br>	InputStream in = new BufferedInputStream(new FileInputStream(inpath)); 
</br>	OutputStream out = fs.append(new Path(hdfs_path)); 
</br>	IOUtils.copyBytes(in, out, 4096, true); 
</br>} catch (IOException e) { 
</br>	System.out.println("--->exception:" + e.getMessage()); 
</br>	e.printStackTrace(); 
</br>} 
</br>} 
</br>}