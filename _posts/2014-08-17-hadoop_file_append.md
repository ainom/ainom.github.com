---
layout: post
date: "2014-08-17 23:49:PM"
title: "Hadoop_File_Append"
description: "Hadoop 0.20之前的版本应该不支持文件追加功能，我用的是1.0版本的。要想使用文件追加写入功能，先配置hdfs-site.xml。"
category: hadoop
tags: [hadoop]
---
{% include JB/setup %}

Hadoop 0.20之前的版本应该不支持文件追加功能，我用的是1.0版本的。要想使用文件追加写入功能，先配置hdfs-site.xml,如下：
     <property>
        <name>dfs.support.append</name>
        <value>true</value>
     </property>
dfs.support.append默认是关闭的。
然后程序打开文件时用：
FileSystem fs = FileSystem.get(URI.create(dst), conf);
FSDataOutputStream out = fs.append(new Path(dst));
在网上找了一天都没有找到解决方法，而且都说hadoop没有这个功能。最后没办法看了下异常，突然发现异常中用个dfs.support.append设置的提示，配置了下，居然成功了，特此分享。



import org.apache.hadoop.conf.Configuration; 
import org.apache.hadoop.fs.FileSystem; 
import org.apache.hadoop.fs.Path; 
import org.apache.hadoop.io.IOUtils; 

import java.io.*; 
import java.net.URI; 
/** 
* 
* @description: 
* @class: AppendContent.java 
* @author <a href="mailto:javamickey@163.com">javamickey</a> 
* @date 2014年8月13日 下午5:39:39 
* @version 1.0 
*/ 
public class AppendContent { 
public static void main(String[] args) { 
	System.out.println("---start merge---"); 
	String hdfs_path = "hdfs://0.0.0.0/tmp/logs/ttk/ttk_show/2014-08-11/ttk_show.log.2014-08-11-01.log";//文件路径 
	Configuration conf = new Configuration(); 
	conf.setBoolean("dfs.support.append", true); 
	conf.setBoolean("dfs.support.broken.append", true); 
	conf.setBoolean("dfs.client.block.write.replace-datanode-on-failure.policy.enable", false); 
	conf.set("dfs.client.block.write.replace-datanode-on-failure.policy", "NEVER"); 
	String inpath = "/home/append.txt"; 
	FileSystem fs = null; 
try { 
	System.out.println("--------------------start-------------------"); 
	fs = FileSystem.get(URI.create(hdfs_path), conf); 
	//要追加的文件流，inpath为文件 
	fs = FileSystem.get(URI.create(hdfs_path), conf); 
	InputStream in = new BufferedInputStream(new FileInputStream(inpath)); 
	OutputStream out = fs.append(new Path(hdfs_path)); 
	IOUtils.copyBytes(in, out, 4096, true); 
} catch (IOException e) { 
	System.out.println("--->exception:" + e.getMessage()); 
	e.printStackTrace(); 
} 
} 
}