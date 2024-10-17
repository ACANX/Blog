---
title: Markdown文档渲染出的代码样式
date: 2018-09-06 10:10:00
categories: "Markdown" #文章分类目录 可以省略
tags: #Java 
     - Markdown
     - 前端
     - JS
description: Markdown代码样式  #
---


# 代码样式



# Java

```java
package com.acanx.thread;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.FutureTask;

/**
 * Created by ACANX on 2018/9/3.
 * MultiThread :
 * <p>
 * <p>
 * 1.继承Thread类，重写该类的run()方法。
 * 2.实现Runnable接口，并重写该接口的run()方法，该run()方法同样是线程执行体，
 * 创建Runnable实现类的实例，并以此实例作为Thread类的target来创建Thread对象，
 * 该Thread对象才是真正的线程对象。
 */
public class MultiThread {

    private static final Logger logger = LoggerFactory.getLogger(MultiThread.class);

    private WhutStudent whutStudent;


    public static void main(String[] args) {
        Thread.currentThread().setName("Main Thread:");
        String str = " 1234566";
        logger.debug("-------程序启动------------------------");

        //在类中的写法  继承Thread类
        Thread threadByThreadInMain = new Thread() {
            int j = 1;
            @Override
            public void run() {
                Thread.currentThread().setName("threadByThreadInMain ");
                for (int j = 0; j < 100; j++) {
                    logger.debug(Thread.currentThread().getName() + " " + j );
                    try {
                        Thread.sleep(100);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    WhutStudent w=WhutStudent.getInstance();
                    logger.debug(w.getStudentId()+" "+Thread.currentThread().getName());
                    w.setStudentId("StudentID:threadByThreadInMain"+j);

                }
            }
        };

        //在类中的写法 实现Runnable接口
        Thread threadByRunnableInMain = new Thread(new Runnable() {
            @Override
            public void run() {
                Thread.currentThread().setName("threadByRunnableInMain ");
                logger.debug("new Thread 2");//输出:new Thread 2
                for (int j = 0; j < 100; j++) {
                    logger.debug(Thread.currentThread().getName() + " " + j + "   " );
                    try {
                        Thread.sleep(100);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    WhutStudent w=WhutStudent.getInstance();
                    logger.debug(w.getStudentId()+" "+Thread.currentThread().getName());
                    w.setStudentId("StudentID:threadByRunnableInMain"+j);

                }
            }
        });

        MyThread myThread = new MyThread();
        myThread.start();

        MyRunnable myRunnable = new MyRunnable();
        Thread myRunnableThread=new Thread(myRunnable);
        myRunnableThread.start();

        threadByThreadInMain.start();


        threadByRunnableInMain.start();



        //执行Callable 方式，需要FutureTask 实现实现，用于接收运算结果
        FutureTask<Integer> futureTask = new FutureTask<Integer>(new MyCallable());
        new Thread(futureTask).start();
        //接收线程运算后的结果
        try {
            Integer sum = futureTask.get();
            logger.debug("sum="+sum);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }


    }
}


```




# Python

```python
# -*- coding: utf-8 -*-
import os
import sys
import urllib2
import requests
import re
from lxml import etree


def StringListSave(save_path, filename, slist):
    if not os.path.exists(save_path):
        os.makedirs(save_path)
    path = save_path+"/"+filename+".txt"
    with open(path, "w+") as fp:
        for s in slist:
            fp.write("%s\t\t%s\n" % (s[0].encode("utf8"), s[1].encode("utf8")))

def Page_Info(myPage):
    '''Regex'''
    mypage_Info = re.findall(r'<div class="titleBar" id=".*?"><h2>(.*?)</h2><div class="more"><a href="(.*?)">.*?</a></div></div>', myPage, re.S)
    return mypage_Info

def New_Page_Info(new_page):
    '''Regex(slowly) or Xpath(fast)'''
    # new_page_Info = re.findall(r'<td class=".*?">.*?<a href="(.*?)\.html".*?>(.*?)</a></td>', new_page, re.S)
    # # new_page_Info = re.findall(r'<td class=".*?">.*?<a href="(.*?)">(.*?)</a></td>', new_page, re.S) # bugs
    # results = []
    # for url, item in new_page_Info:
    #     results.append((item, url+".html"))
    # return results
    dom = etree.HTML(new_page)
    new_items = dom.xpath('//tr/td/a/text()')
    new_urls = dom.xpath('//tr/td/a/@href')
    assert(len(new_items) == len(new_urls))
    return zip(new_items, new_urls)

def Spider(url):
    i = 0
    print "downloading ", url
    myPage = requests.get(url).content.decode("gbk")
    # myPage = urllib2.urlopen(url).read().decode("gbk")
    myPageResults = Page_Info(myPage)
    save_path = u"网易新闻抓取"
    filename = str(i)+"_"+u"新闻排行榜"
    StringListSave(save_path, filename, myPageResults)
    i += 1
    for item, url in myPageResults:
        print "downloading ", url
        new_page = requests.get(url).content.decode("gbk")
        # new_page = urllib2.urlopen(url).read().decode("gbk")
        newPageResults = New_Page_Info(new_page)
        filename = str(i)+"_"+item
        StringListSave(save_path, filename, newPageResults)
        i += 1


if __name__ == '__main__':
    print "start"
    start_url = "http://news.163.com/rank/"
    Spider(start_url)
    print "end"

```

# PHP
```php
<?php
namespace PhpOffice\PhpSpreadsheet;
use PhpOffice\PhpSpreadsheet\Shared\File;
/**
 * Factory to create readers and writers easily.
 *
 * It is not required to use this class, but it should make it easier to read and write files.
 * Especially for reading files with an unknown format.
 */
abstract class IOFactory
{
    private static $readers = [
        'Xlsx' => Reader\Xlsx::class,
        'Xls' => Reader\Xls::class,
        'Xml' => Reader\Xml::class,
        'Ods' => Reader\Ods::class,
        'Slk' => Reader\Slk::class,
        'Gnumeric' => Reader\Gnumeric::class,
        'Html' => Reader\Html::class,
        'Csv' => Reader\Csv::class,
    ];
    private static $writers = [
        'Xls' => Writer\Xls::class,
        'Xlsx' => Writer\Xlsx::class,
        'Ods' => Writer\Ods::class,
        'Csv' => Writer\Csv::class,
        'Html' => Writer\Html::class,
        'Tcpdf' => Writer\Pdf\Tcpdf::class,
        'Dompdf' => Writer\Pdf\Dompdf::class,
        'Mpdf' => Writer\Pdf\Mpdf::class,
    ];
    /**
     * Create Writer\IWriter.
     *
     * @param Spreadsheet $spreadsheet
     * @param string $writerType Example: Xlsx
     *
     * @throws Writer\Exception
     *
     * @return Writer\IWriter
     */
    public static function createWriter(Spreadsheet $spreadsheet, $writerType)
    {
        if (!isset(self::$writers[$writerType])) {
            throw new Writer\Exception("No writer found for type $writerType");
        }
        // Instantiate writer
        $className = self::$writers[$writerType];
        $writer = new $className($spreadsheet);
        return $writer;
    }
    /**
     * Create Reader\IReader.
     *
     * @param string $readerType Example: Xlsx
     *
     * @throws Reader\Exception
     *
     * @return Reader\IReader
     */
    public static function createReader($readerType)
    {
        if (!isset(self::$readers[$readerType])) {
            throw new Reader\Exception("No reader found for type $readerType");
        }
        // Instantiate reader
        $className = self::$readers[$readerType];
        $reader = new $className();
        return $reader;
    }
    /**
     * Loads Spreadsheet from file using automatic Reader\IReader resolution.
     *
     * @param string $pFilename The name of the spreadsheet file
     *
     * @throws Reader\Exception
     *
     * @return Spreadsheet
     */
    public static function load($pFilename)
    {
        $reader = self::createReaderForFile($pFilename);
        return $reader->load($pFilename);
    }
    /**
     * Identify file type using automatic Reader\IReader resolution.
     *
     * @param string $pFilename The name of the spreadsheet file to identify
     *
     * @throws Reader\Exception
     *
     * @return string
     */
    public static function identify($pFilename)
    {
        $reader = self::createReaderForFile($pFilename);
        $className = get_class($reader);
        $classType = explode('\\', $className);
        unset($reader);
        return array_pop($classType);
    }
    /**
     * Create Reader\IReader for file using automatic Reader\IReader resolution.
     *
     * @param string $filename The name of the spreadsheet file
     *
     * @throws Reader\Exception
     *
     * @return Reader\IReader
     */
    public static function createReaderForFile($filename)
    {
        File::assertFile($filename);
        // First, lucky guess by inspecting file extension
        $guessedReader = self::getReaderTypeFromExtension($filename);
        if ($guessedReader !== null) {
            $reader = self::createReader($guessedReader);
            // Let's see if we are lucky
            if (isset($reader) && $reader->canRead($filename)) {
                return $reader;
            }
        }
        // If we reach here then "lucky guess" didn't give any result
        // Try walking through all the options in self::$autoResolveClasses
        foreach (self::$readers as $type => $class) {
            //    Ignore our original guess, we know that won't work
            if ($type !== $guessedReader) {
                $reader = self::createReader($type);
                if ($reader->canRead($filename)) {
                    return $reader;
                }
            }
        }
        throw new Reader\Exception('Unable to identify a reader for this file');
    }
    /**
     * Guess a reader type from the file extension, if any.
     *
     * @param string $filename
     *
     * @return null|string
     */
    private static function getReaderTypeFromExtension($filename)
    {
        $pathinfo = pathinfo($filename);
        if (!isset($pathinfo['extension'])) {
            return null;
        }
        switch (strtolower($pathinfo['extension'])) {
            case 'xlsx': // Excel (OfficeOpenXML) Spreadsheet
            case 'xlsm': // Excel (OfficeOpenXML) Macro Spreadsheet (macros will be discarded)
            case 'xltx': // Excel (OfficeOpenXML) Template
            case 'xltm': // Excel (OfficeOpenXML) Macro Template (macros will be discarded)
                return 'Xlsx';
            case 'xls': // Excel (BIFF) Spreadsheet
            case 'xlt': // Excel (BIFF) Template
                return 'Xls';
            case 'ods': // Open/Libre Offic Calc
            case 'ots': // Open/Libre Offic Calc Template
                return 'Ods';
            case 'slk':
                return 'Slk';
            case 'xml': // Excel 2003 SpreadSheetML
                return 'Xml';
            case 'gnumeric':
                return 'Gnumeric';
            case 'htm':
            case 'html':
                return 'Html';
            case 'csv':
                // Do nothing
                // We must not try to use CSV reader since it loads
                // all files including Excel files etc.
                return null;
            default:
                return null;
        }
    }
    /**
     * Register a writer with its type and class name.
     *
     * @param string $writerType
     * @param string $writerClass
     */
    public static function registerWriter($writerType, $writerClass)
    {
        if (!is_a($writerClass, Writer\IWriter::class, true)) {
            throw new Writer\Exception('Registered writers must implement ' . Writer\IWriter::class);
        }
        self::$writers[$writerType] = $writerClass;
    }
    /**
     * Register a reader with its type and class name.
     *
     * @param string $readerType
     * @param string $readerClass
     */
    public static function registerReader($readerType, $readerClass)
    {
        if (!is_a($readerClass, Reader\IReader::class, true)) {
            throw new Reader\Exception('Registered readers must implement ' . Reader\IReader::class);
        }
        self::$readers[$readerType] = $readerClass;
    }
}
```



# Go
```go
package list

// Element is an element of a linked list.
type Element struct {
	// Next and previous pointers in the doubly-linked list of elements.
	// To simplify the implementation, internally a list l is implemented
	// as a ring, such that &l.root is both the next element of the last
	// list element (l.Back()) and the previous element of the first list
	// element (l.Front()).
	next, prev *Element

	// The list to which this element belongs.
	list *List

	// The value stored with this element.
	Value interface{}
}

// Next returns the next list element or nil.
func (e *Element) Next() *Element {
	if p := e.next; e.list != nil && p != &e.list.root {
		return p
	}
	return nil
}

// Prev returns the previous list element or nil.
func (e *Element) Prev() *Element {
	if p := e.prev; e.list != nil && p != &e.list.root {
		return p
	}
	return nil
}

// List represents a doubly linked list.
// The zero value for List is an empty list ready to use.
type List struct {
	root Element // sentinel list element, only &root, root.prev, and root.next are used
	len  int     // current list length excluding (this) sentinel element
}

// Init initializes or clears list l.
func (l *List) Init() *List {
	l.root.next = &l.root
	l.root.prev = &l.root
	l.len = 0
	return l
}

// New returns an initialized list.
func New() *List { return new(List).Init() }

// Len returns the number of elements of list l.
// The complexity is O(1).
func (l *List) Len() int { return l.len }

// Front returns the first element of list l or nil if the list is empty.
func (l *List) Front() *Element {
	if l.len == 0 {
		return nil
	}
	return l.root.next
}

// Back returns the last element of list l or nil if the list is empty.
func (l *List) Back() *Element {
	if l.len == 0 {
		return nil
	}
	return l.root.prev
}

// lazyInit lazily initializes a zero List value.
func (l *List) lazyInit() {
	if l.root.next == nil {
		l.Init()
	}
}

// insert inserts e after at, increments l.len, and returns e.
func (l *List) insert(e, at *Element) *Element {
	n := at.next
	at.next = e
	e.prev = at
	e.next = n
	n.prev = e
	e.list = l
	l.len++
	return e
}

// insertValue is a convenience wrapper for insert(&Element{Value: v}, at).
func (l *List) insertValue(v interface{}, at *Element) *Element {
	return l.insert(&Element{Value: v}, at)
}

// remove removes e from its list, decrements l.len, and returns e.
func (l *List) remove(e *Element) *Element {
	e.prev.next = e.next
	e.next.prev = e.prev
	e.next = nil // avoid memory leaks
	e.prev = nil // avoid memory leaks
	e.list = nil
	l.len--
	return e
}

// Remove removes e from l if e is an element of list l.
// It returns the element value e.Value.
// The element must not be nil.
func (l *List) Remove(e *Element) interface{} {
	if e.list == l {
		// if e.list == l, l must have been initialized when e was inserted
		// in l or l == nil (e is a zero Element) and l.remove will crash
		l.remove(e)
	}
	return e.Value
}

// PushFront inserts a new element e with value v at the front of list l and returns e.
func (l *List) PushFront(v interface{}) *Element {
	l.lazyInit()
	return l.insertValue(v, &l.root)
}

// PushBack inserts a new element e with value v at the back of list l and returns e.
func (l *List) PushBack(v interface{}) *Element {
	l.lazyInit()
	return l.insertValue(v, l.root.prev)
}

// InsertBefore inserts a new element e with value v immediately before mark and returns e.
// If mark is not an element of l, the list is not modified.
// The mark must not be nil.
func (l *List) InsertBefore(v interface{}, mark *Element) *Element {
	if mark.list != l {
		return nil
	}
	// see comment in List.Remove about initialization of l
	return l.insertValue(v, mark.prev)
}

// InsertAfter inserts a new element e with value v immediately after mark and returns e.
// If mark is not an element of l, the list is not modified.
// The mark must not be nil.
func (l *List) InsertAfter(v interface{}, mark *Element) *Element {
	if mark.list != l {
		return nil
	}
	// see comment in List.Remove about initialization of l
	return l.insertValue(v, mark)
}

// MoveToFront moves element e to the front of list l.
// If e is not an element of l, the list is not modified.
// The element must not be nil.
func (l *List) MoveToFront(e *Element) {
	if e.list != l || l.root.next == e {
		return
	}
	// see comment in List.Remove about initialization of l
	l.insert(l.remove(e), &l.root)
}

// MoveToBack moves element e to the back of list l.
// If e is not an element of l, the list is not modified.
// The element must not be nil.
func (l *List) MoveToBack(e *Element) {
	if e.list != l || l.root.prev == e {
		return
	}
	// see comment in List.Remove about initialization of l
	l.insert(l.remove(e), l.root.prev)
}

// MoveBefore moves element e to its new position before mark.
// If e or mark is not an element of l, or e == mark, the list is not modified.
// The element and mark must not be nil.
func (l *List) MoveBefore(e, mark *Element) {
	if e.list != l || e == mark || mark.list != l {
		return
	}
	l.insert(l.remove(e), mark.prev)
}

// MoveAfter moves element e to its new position after mark.
// If e or mark is not an element of l, or e == mark, the list is not modified.
// The element and mark must not be nil.
func (l *List) MoveAfter(e, mark *Element) {
	if e.list != l || e == mark || mark.list != l {
		return
	}
	l.insert(l.remove(e), mark)
}

// PushBackList inserts a copy of an other list at the back of list l.
// The lists l and other may be the same. They must not be nil.
func (l *List) PushBackList(other *List) {
	l.lazyInit()
	for i, e := other.Len(), other.Front(); i > 0; i, e = i-1, e.Next() {
		l.insertValue(e.Value, l.root.prev)
	}
}

// PushFrontList inserts a copy of an other list at the front of list l.
// The lists l and other may be the same. They must not be nil.
func (l *List) PushFrontList(other *List) {
	l.lazyInit()
	for i, e := other.Len(), other.Back(); i > 0; i, e = i-1, e.Prev() {
		l.insertValue(e.Value, &l.root)
	}
}
```


# C++
```c++
#include <grpc/support/time.h>
#include <grpcpp/support/config.h>
#include <grpcpp/support/time.h>

using std::chrono::duration_cast;
using std::chrono::high_resolution_clock;
using std::chrono::nanoseconds;
using std::chrono::seconds;
using std::chrono::system_clock;

namespace grpc {

void Timepoint2Timespec(const system_clock::time_point& from,
                        gpr_timespec* to) {
  system_clock::duration deadline = from.time_since_epoch();
  seconds secs = duration_cast<seconds>(deadline);
  if (from == system_clock::time_point::max() ||
      secs.count() >= gpr_inf_future(GPR_CLOCK_REALTIME).tv_sec ||
      secs.count() < 0) {
    *to = gpr_inf_future(GPR_CLOCK_REALTIME);
    return;
  }
  nanoseconds nsecs = duration_cast<nanoseconds>(deadline - secs);
  to->tv_sec = static_cast<int64_t>(secs.count());
  to->tv_nsec = static_cast<int32_t>(nsecs.count());
  to->clock_type = GPR_CLOCK_REALTIME;
}

void TimepointHR2Timespec(const high_resolution_clock::time_point& from,
                          gpr_timespec* to) {
  high_resolution_clock::duration deadline = from.time_since_epoch();
  seconds secs = duration_cast<seconds>(deadline);
  if (from == high_resolution_clock::time_point::max() ||
      secs.count() >= gpr_inf_future(GPR_CLOCK_REALTIME).tv_sec ||
      secs.count() < 0) {
    *to = gpr_inf_future(GPR_CLOCK_REALTIME);
    return;
  }
  nanoseconds nsecs = duration_cast<nanoseconds>(deadline - secs);
  to->tv_sec = static_cast<int64_t>(secs.count());
  to->tv_nsec = static_cast<int32_t>(nsecs.count());
  to->clock_type = GPR_CLOCK_REALTIME;
}

system_clock::time_point Timespec2Timepoint(gpr_timespec t) {
  if (gpr_time_cmp(t, gpr_inf_future(t.clock_type)) == 0) {
    return system_clock::time_point::max();
  }
  t = gpr_convert_clock_type(t, GPR_CLOCK_REALTIME);
  system_clock::time_point tp;
  tp += duration_cast<system_clock::time_point::duration>(seconds(t.tv_sec));
  tp +=
      duration_cast<system_clock::time_point::duration>(nanoseconds(t.tv_nsec));
  return tp;
}

}  // namespace grpc
```


# C
```c
#include <stdio.h>
#include <stdlib.h>
#include <time.h>

void shellSort(int array[], int value){
  int i = value;
  int j, k, tmp;
  for (i = value / 2; i > 0; i = i / 2){
    for (j = i; j < value; j++){
      for(k = j - i; k >= 0; k = k - i){
        if (array[k+i] >= array[k]){
          break;
        }
        else{
          tmp = array[k];
          array[k] = array[k+i];
          array[k+i] = tmp;
        }
      }
    }
  }
}


int main(){
    int array[20];
    int range = 500;
    for(int i = 0; i < 100; i++){
     array[i] = rand() % range + 1;
    }
    int size = sizeof array / sizeof array[0];



    clock_t start = clock();
    shellSort(array,size);
    clock_t end = clock();
    double time_spent = (double)(end - start) / CLOCKS_PER_SEC;


    printf("Data Sorted\n");
    printf("%s\n", "Shell Sort Big O Notation:\n--> Best Case: O(n log(n))\n--> Average Case: depends on gap sequence\n--> Worst Case: O(n)\n");
    printf("Time spent sorting: %f\n", time_spent);

  return 0;
}
```



# C\#    #  C#
```csharp
#pragma warning disable 1591, 0612, 3021
#region Designer generated code

using pb = global::Google.Protobuf;
using pbc = global::Google.Protobuf.Collections;
using pbr = global::Google.Protobuf.Reflection;
using scg = global::System.Collections.Generic;
namespace Math {

  /// <summary>Holder for reflection information generated from math/math.proto</summary>
  public static partial class MathReflection {

    #region Descriptor
    /// <summary>File descriptor for math/math.proto</summary>
    public static pbr::FileDescriptor Descriptor {
      get { return descriptor; }
    }
    private static pbr::FileDescriptor descriptor;

    static MathReflection() {
      byte[] descriptorData = global::System.Convert.FromBase64String(
          string.Concat(
            "Cg9tYXRoL21hdGgucHJvdG8SBG1hdGgiLAoHRGl2QXJncxIQCghkaXZpZGVu",
            "ZBgBIAEoAxIPCgdkaXZpc29yGAIgASgDIi8KCERpdlJlcGx5EhAKCHF1b3Rp",
            "ZW50GAEgASgDEhEKCXJlbWFpbmRlchgCIAEoAyIYCgdGaWJBcmdzEg0KBWxp",
            "bWl0GAEgASgDIhIKA051bRILCgNudW0YASABKAMiGQoIRmliUmVwbHkSDQoF",
            "Y291bnQYASABKAMypAEKBE1hdGgSJgoDRGl2Eg0ubWF0aC5EaXZBcmdzGg4u",
            "bWF0aC5EaXZSZXBseSIAEi4KB0Rpdk1hbnkSDS5tYXRoLkRpdkFyZ3MaDi5t",
            "YXRoLkRpdlJlcGx5IgAoATABEiMKA0ZpYhINLm1hdGguRmliQXJncxoJLm1h",
            "dGguTnVtIgAwARIfCgNTdW0SCS5tYXRoLk51bRoJLm1hdGguTnVtIgAoAWIG",
            "cHJvdG8z"));
      descriptor = pbr::FileDescriptor.FromGeneratedCode(descriptorData,
          new pbr::FileDescriptor[] { },
          new pbr::GeneratedClrTypeInfo(null, new pbr::GeneratedClrTypeInfo[] {
            new pbr::GeneratedClrTypeInfo(typeof(global::Math.DivArgs), global::Math.DivArgs.Parser, new[]{ "Dividend", "Divisor" }, null, null, null),
            new pbr::GeneratedClrTypeInfo(typeof(global::Math.DivReply), global::Math.DivReply.Parser, new[]{ "Quotient", "Remainder" }, null, null, null),
            new pbr::GeneratedClrTypeInfo(typeof(global::Math.FibArgs), global::Math.FibArgs.Parser, new[]{ "Limit" }, null, null, null),
            new pbr::GeneratedClrTypeInfo(typeof(global::Math.Num), global::Math.Num.Parser, new[]{ "Num_" }, null, null, null),
            new pbr::GeneratedClrTypeInfo(typeof(global::Math.FibReply), global::Math.FibReply.Parser, new[]{ "Count" }, null, null, null)
          }));
    }
    #endregion

  }
  #region Messages
  public sealed partial class DivArgs : pb::IMessage<DivArgs> {
    private static readonly pb::MessageParser<DivArgs> _parser = new pb::MessageParser<DivArgs>(() => new DivArgs());
    private pb::UnknownFieldSet _unknownFields;
    [global::System.Diagnostics.DebuggerNonUserCodeAttribute]
    public static pb::MessageParser<DivArgs> Parser { get { return _parser; } }

    [global::System.Diagnostics.DebuggerNonUserCodeAttribute]
    public static pbr::MessageDescriptor Descriptor {
      get { return global::Math.MathReflection.Descriptor.MessageTypes[0]; }
    }

    [global::System.Diagnostics.DebuggerNonUserCodeAttribute]
    pbr::MessageDescriptor pb::IMessage.Descriptor {
      get { return Descriptor; }
    }

    [global::System.Diagnostics.DebuggerNonUserCodeAttribute]
    public DivArgs() {
      OnConstruction();
    }

    partial void OnConstruction();

    [global::System.Diagnostics.DebuggerNonUserCodeAttribute]
    public DivArgs(DivArgs other) : this() {
      dividend_ = other.dividend_;
      divisor_ = other.divisor_;
      _unknownFields = pb::UnknownFieldSet.Clone(other._unknownFields);
    }

    [global::System.Diagnostics.DebuggerNonUserCodeAttribute]
    public DivArgs Clone() {
      return new DivArgs(this);
    }

    /// <summary>Field number for the "dividend" field.</summary>
    public const int DividendFieldNumber = 1;
    private long dividend_;
    [global::System.Diagnostics.DebuggerNonUserCodeAttribute]
    public long Dividend {
      get { return dividend_; }
      set {
        dividend_ = value;
      }
    }

    /// <summary>Field number for the "divisor" field.</summary>
    public const int DivisorFieldNumber = 2;
    private long divisor_;
    [global::System.Diagnostics.DebuggerNonUserCodeAttribute]
    public long Divisor {
      get { return divisor_; }
      set {
        divisor_ = value;
      }
    }

    [global::System.Diagnostics.DebuggerNonUserCodeAttribute]
    public override bool Equals(object other) {
      return Equals(other as DivArgs);
    }

    [global::System.Diagnostics.DebuggerNonUserCodeAttribute]
    public bool Equals(DivArgs other) {
      if (ReferenceEquals(other, null)) {
        return false;
      }
      if (ReferenceEquals(other, this)) {
        return true;
      }
      if (Dividend != other.Dividend) return false;
      if (Divisor != other.Divisor) return false;
      return Equals(_unknownFields, other._unknownFields);
    }

    [global::System.Diagnostics.DebuggerNonUserCodeAttribute]
    public override int GetHashCode() {
      int hash = 1;
      if (Dividend != 0L) hash ^= Dividend.GetHashCode();
      if (Divisor != 0L) hash ^= Divisor.GetHashCode();
      if (_unknownFields != null) {
        hash ^= _unknownFields.GetHashCode();
      }
      return hash;
    }

    [global::System.Diagnostics.DebuggerNonUserCodeAttribute]
    public override string ToString() {
      return pb::JsonFormatter.ToDiagnosticString(this);
    }

    [global::System.Diagnostics.DebuggerNonUserCodeAttribute]
    public void WriteTo(pb::CodedOutputStream output) {
      if (Dividend != 0L) {
        output.WriteRawTag(8);
        output.WriteInt64(Dividend);
      }
      if (Divisor != 0L) {
        output.WriteRawTag(16);
        output.WriteInt64(Divisor);
      }
      if (_unknownFields != null) {
        _unknownFields.WriteTo(output);
      }
    }

    [global::System.Diagnostics.DebuggerNonUserCodeAttribute]
    public int CalculateSize() {
      int size = 0;
      if (Dividend != 0L) {
        size += 1 + pb::CodedOutputStream.ComputeInt64Size(Dividend);
      }
      if (Divisor != 0L) {
        size += 1 + pb::CodedOutputStream.ComputeInt64Size(Divisor);
      }
      if (_unknownFields != null) {
        size += _unknownFields.CalculateSize();
      }
      return size;
    }

    [global::System.Diagnostics.DebuggerNonUserCodeAttribute]
    public void MergeFrom(DivArgs other) {
      if (other == null) {
        return;
      }
      if (other.Dividend != 0L) {
        Dividend = other.Dividend;
      }
      if (other.Divisor != 0L) {
        Divisor = other.Divisor;
      }
      _unknownFields = pb::UnknownFieldSet.MergeFrom(_unknownFields, other._unknownFields);
    }

    [global::System.Diagnostics.DebuggerNonUserCodeAttribute]
    public void MergeFrom(pb::CodedInputStream input) {
      uint tag;
      while ((tag = input.ReadTag()) != 0) {
        switch(tag) {
          default:
            _unknownFields = pb::UnknownFieldSet.MergeFieldFrom(_unknownFields, input);
            break;
          case 8: {
            Dividend = input.ReadInt64();
            break;
          }
          case 16: {
            Divisor = input.ReadInt64();
            break;
          }
        }
      }
    }

  }

  public sealed partial class DivReply : pb::IMessage<DivReply> {
    private static readonly pb::MessageParser<DivReply> _parser = new pb::MessageParser<DivReply>(() => new DivReply());
    private pb::UnknownFieldSet _unknownFields;
    [global::System.Diagnostics.DebuggerNonUserCodeAttribute]
    public static pb::MessageParser<DivReply> Parser { get { return _parser; } }

    [global::System.Diagnostics.DebuggerNonUserCodeAttribute]
    public static pbr::MessageDescriptor Descriptor {
      get { return global::Math.MathReflection.Descriptor.MessageTypes[1]; }
    }

    [global::System.Diagnostics.DebuggerNonUserCodeAttribute]
    pbr::MessageDescriptor pb::IMessage.Descriptor {
      get { return Descriptor; }
    }

    [global::System.Diagnostics.DebuggerNonUserCodeAttribute]
    public DivReply() {
      OnConstruction();
    }

    partial void OnConstruction();

    [global::System.Diagnostics.DebuggerNonUserCodeAttribute]
    public DivReply(DivReply other) : this() {
      quotient_ = other.quotient_;
      remainder_ = other.remainder_;
      _unknownFields = pb::UnknownFieldSet.Clone(other._unknownFields);
    }

    [global::System.Diagnostics.DebuggerNonUserCodeAttribute]
    public DivReply Clone() {
      return new DivReply(this);
    }

    /// <summary>Field number for the "quotient" field.</summary>
    public const int QuotientFieldNumber = 1;
    private long quotient_;
    [global::System.Diagnostics.DebuggerNonUserCodeAttribute]
    public long Quotient {
      get { return quotient_; }
      set {
        quotient_ = value;
      }
    }

    /// <summary>Field number for the "remainder" field.</summary>
    public const int RemainderFieldNumber = 2;
    private long remainder_;
    [global::System.Diagnostics.DebuggerNonUserCodeAttribute]
    public long Remainder {
      get { return remainder_; }
      set {
        remainder_ = value;
      }
    }

    [global::System.Diagnostics.DebuggerNonUserCodeAttribute]
    public override bool Equals(object other) {
      return Equals(other as DivReply);
    }

    [global::System.Diagnostics.DebuggerNonUserCodeAttribute]
    public bool Equals(DivReply other) {
      if (ReferenceEquals(other, null)) {
        return false;
      }
      if (ReferenceEquals(other, this)) {
        return true;
      }
      if (Quotient != other.Quotient) return false;
      if (Remainder != other.Remainder) return false;
      return Equals(_unknownFields, other._unknownFields);
    }

    [global::System.Diagnostics.DebuggerNonUserCodeAttribute]
    public override int GetHashCode() {
      int hash = 1;
      if (Quotient != 0L) hash ^= Quotient.GetHashCode();
      if (Remainder != 0L) hash ^= Remainder.GetHashCode();
      if (_unknownFields != null) {
        hash ^= _unknownFields.GetHashCode();
      }
      return hash;
    }

    [global::System.Diagnostics.DebuggerNonUserCodeAttribute]
    public override string ToString() {
      return pb::JsonFormatter.ToDiagnosticString(this);
    }

    [global::System.Diagnostics.DebuggerNonUserCodeAttribute]
    public void WriteTo(pb::CodedOutputStream output) {
      if (Quotient != 0L) {
        output.WriteRawTag(8);
        output.WriteInt64(Quotient);
      }
      if (Remainder != 0L) {
        output.WriteRawTag(16);
        output.WriteInt64(Remainder);
      }
      if (_unknownFields != null) {
        _unknownFields.WriteTo(output);
      }
    }

    [global::System.Diagnostics.DebuggerNonUserCodeAttribute]
    public int CalculateSize() {
      int size = 0;
      if (Quotient != 0L) {
        size += 1 + pb::CodedOutputStream.ComputeInt64Size(Quotient);
      }
      if (Remainder != 0L) {
        size += 1 + pb::CodedOutputStream.ComputeInt64Size(Remainder);
      }
      if (_unknownFields != null) {
        size += _unknownFields.CalculateSize();
      }
      return size;
    }

    [global::System.Diagnostics.DebuggerNonUserCodeAttribute]
    public void MergeFrom(DivReply other) {
      if (other == null) {
        return;
      }
      if (other.Quotient != 0L) {
        Quotient = other.Quotient;
      }
      if (other.Remainder != 0L) {
        Remainder = other.Remainder;
      }
      _unknownFields = pb::UnknownFieldSet.MergeFrom(_unknownFields, other._unknownFields);
    }

    [global::System.Diagnostics.DebuggerNonUserCodeAttribute]
    public void MergeFrom(pb::CodedInputStream input) {
      uint tag;
      while ((tag = input.ReadTag()) != 0) {
        switch(tag) {
          default:
            _unknownFields = pb::UnknownFieldSet.MergeFieldFrom(_unknownFields, input);
            break;
          case 8: {
            Quotient = input.ReadInt64();
            break;
          }
          case 16: {
            Remainder = input.ReadInt64();
            break;
          }
        }
      }
    }

  }

  public sealed partial class FibArgs : pb::IMessage<FibArgs> {
    private static readonly pb::MessageParser<FibArgs> _parser = new pb::MessageParser<FibArgs>(() => new FibArgs());
    private pb::UnknownFieldSet _unknownFields;
    [global::System.Diagnostics.DebuggerNonUserCodeAttribute]
    public static pb::MessageParser<FibArgs> Parser { get { return _parser; } }

    [global::System.Diagnostics.DebuggerNonUserCodeAttribute]
    public static pbr::MessageDescriptor Descriptor {
      get { return global::Math.MathReflection.Descriptor.MessageTypes[2]; }
    }

    [global::System.Diagnostics.DebuggerNonUserCodeAttribute]
    pbr::MessageDescriptor pb::IMessage.Descriptor {
      get { return Descriptor; }
    }

    [global::System.Diagnostics.DebuggerNonUserCodeAttribute]
    public FibArgs() {
      OnConstruction();
    }

    partial void OnConstruction();

    [global::System.Diagnostics.DebuggerNonUserCodeAttribute]
    public FibArgs(FibArgs other) : this() {
      limit_ = other.limit_;
      _unknownFields = pb::UnknownFieldSet.Clone(other._unknownFields);
    }

    [global::System.Diagnostics.DebuggerNonUserCodeAttribute]
    public FibArgs Clone() {
      return new FibArgs(this);
    }

    /// <summary>Field number for the "limit" field.</summary>
    public const int LimitFieldNumber = 1;
    private long limit_;
    [global::System.Diagnostics.DebuggerNonUserCodeAttribute]
    public long Limit {
      get { return limit_; }
      set {
        limit_ = value;
      }
    }

    [global::System.Diagnostics.DebuggerNonUserCodeAttribute]
    public override bool Equals(object other) {
      return Equals(other as FibArgs);
    }

    [global::System.Diagnostics.DebuggerNonUserCodeAttribute]
    public bool Equals(FibArgs other) {
      if (ReferenceEquals(other, null)) {
        return false;
      }
      if (ReferenceEquals(other, this)) {
        return true;
      }
      if (Limit != other.Limit) return false;
      return Equals(_unknownFields, other._unknownFields);
    }

    [global::System.Diagnostics.DebuggerNonUserCodeAttribute]
    public override int GetHashCode() {
      int hash = 1;
      if (Limit != 0L) hash ^= Limit.GetHashCode();
      if (_unknownFields != null) {
        hash ^= _unknownFields.GetHashCode();
      }
      return hash;
    }

    [global::System.Diagnostics.DebuggerNonUserCodeAttribute]
    public override string ToString() {
      return pb::JsonFormatter.ToDiagnosticString(this);
    }

    [global::System.Diagnostics.DebuggerNonUserCodeAttribute]
    public void WriteTo(pb::CodedOutputStream output) {
      if (Limit != 0L) {
        output.WriteRawTag(8);
        output.WriteInt64(Limit);
      }
      if (_unknownFields != null) {
        _unknownFields.WriteTo(output);
      }
    }

    [global::System.Diagnostics.DebuggerNonUserCodeAttribute]
    public int CalculateSize() {
      int size = 0;
      if (Limit != 0L) {
        size += 1 + pb::CodedOutputStream.ComputeInt64Size(Limit);
      }
      if (_unknownFields != null) {
        size += _unknownFields.CalculateSize();
      }
      return size;
    }

    [global::System.Diagnostics.DebuggerNonUserCodeAttribute]
    public void MergeFrom(FibArgs other) {
      if (other == null) {
        return;
      }
      if (other.Limit != 0L) {
        Limit = other.Limit;
      }
      _unknownFields = pb::UnknownFieldSet.MergeFrom(_unknownFields, other._unknownFields);
    }

    [global::System.Diagnostics.DebuggerNonUserCodeAttribute]
    public void MergeFrom(pb::CodedInputStream input) {
      uint tag;
      while ((tag = input.ReadTag()) != 0) {
        switch(tag) {
          default:
            _unknownFields = pb::UnknownFieldSet.MergeFieldFrom(_unknownFields, input);
            break;
          case 8: {
            Limit = input.ReadInt64();
            break;
          }
        }
      }
    }

  }

  public sealed partial class Num : pb::IMessage<Num> {
    private static readonly pb::MessageParser<Num> _parser = new pb::MessageParser<Num>(() => new Num());
    private pb::UnknownFieldSet _unknownFields;
    [global::System.Diagnostics.DebuggerNonUserCodeAttribute]
    public static pb::MessageParser<Num> Parser { get { return _parser; } }

    [global::System.Diagnostics.DebuggerNonUserCodeAttribute]
    public static pbr::MessageDescriptor Descriptor {
      get { return global::Math.MathReflection.Descriptor.MessageTypes[3]; }
    }

    [global::System.Diagnostics.DebuggerNonUserCodeAttribute]
    pbr::MessageDescriptor pb::IMessage.Descriptor {
      get { return Descriptor; }
    }

    [global::System.Diagnostics.DebuggerNonUserCodeAttribute]
    public Num() {
      OnConstruction();
    }

    partial void OnConstruction();

    [global::System.Diagnostics.DebuggerNonUserCodeAttribute]
    public Num(Num other) : this() {
      num_ = other.num_;
      _unknownFields = pb::UnknownFieldSet.Clone(other._unknownFields);
    }

    [global::System.Diagnostics.DebuggerNonUserCodeAttribute]
    public Num Clone() {
      return new Num(this);
    }

    /// <summary>Field number for the "num" field.</summary>
    public const int Num_FieldNumber = 1;
    private long num_;
    [global::System.Diagnostics.DebuggerNonUserCodeAttribute]
    public long Num_ {
      get { return num_; }
      set {
        num_ = value;
      }
    }

    [global::System.Diagnostics.DebuggerNonUserCodeAttribute]
    public override bool Equals(object other) {
      return Equals(other as Num);
    }

    [global::System.Diagnostics.DebuggerNonUserCodeAttribute]
    public bool Equals(Num other) {
      if (ReferenceEquals(other, null)) {
        return false;
      }
      if (ReferenceEquals(other, this)) {
        return true;
      }
      if (Num_ != other.Num_) return false;
      return Equals(_unknownFields, other._unknownFields);
    }

    [global::System.Diagnostics.DebuggerNonUserCodeAttribute]
    public override int GetHashCode() {
      int hash = 1;
      if (Num_ != 0L) hash ^= Num_.GetHashCode();
      if (_unknownFields != null) {
        hash ^= _unknownFields.GetHashCode();
      }
      return hash;
    }

    [global::System.Diagnostics.DebuggerNonUserCodeAttribute]
    public override string ToString() {
      return pb::JsonFormatter.ToDiagnosticString(this);
    }

    [global::System.Diagnostics.DebuggerNonUserCodeAttribute]
    public void WriteTo(pb::CodedOutputStream output) {
      if (Num_ != 0L) {
        output.WriteRawTag(8);
        output.WriteInt64(Num_);
      }
      if (_unknownFields != null) {
        _unknownFields.WriteTo(output);
      }
    }

    [global::System.Diagnostics.DebuggerNonUserCodeAttribute]
    public int CalculateSize() {
      int size = 0;
      if (Num_ != 0L) {
        size += 1 + pb::CodedOutputStream.ComputeInt64Size(Num_);
      }
      if (_unknownFields != null) {
        size += _unknownFields.CalculateSize();
      }
      return size;
    }

    [global::System.Diagnostics.DebuggerNonUserCodeAttribute]
    public void MergeFrom(Num other) {
      if (other == null) {
        return;
      }
      if (other.Num_ != 0L) {
        Num_ = other.Num_;
      }
      _unknownFields = pb::UnknownFieldSet.MergeFrom(_unknownFields, other._unknownFields);
    }

    [global::System.Diagnostics.DebuggerNonUserCodeAttribute]
    public void MergeFrom(pb::CodedInputStream input) {
      uint tag;
      while ((tag = input.ReadTag()) != 0) {
        switch(tag) {
          default:
            _unknownFields = pb::UnknownFieldSet.MergeFieldFrom(_unknownFields, input);
            break;
          case 8: {
            Num_ = input.ReadInt64();
            break;
          }
        }
      }
    }

  }

  public sealed partial class FibReply : pb::IMessage<FibReply> {
    private static readonly pb::MessageParser<FibReply> _parser = new pb::MessageParser<FibReply>(() => new FibReply());
    private pb::UnknownFieldSet _unknownFields;
    [global::System.Diagnostics.DebuggerNonUserCodeAttribute]
    public static pb::MessageParser<FibReply> Parser { get { return _parser; } }

    [global::System.Diagnostics.DebuggerNonUserCodeAttribute]
    public static pbr::MessageDescriptor Descriptor {
      get { return global::Math.MathReflection.Descriptor.MessageTypes[4]; }
    }

    [global::System.Diagnostics.DebuggerNonUserCodeAttribute]
    pbr::MessageDescriptor pb::IMessage.Descriptor {
      get { return Descriptor; }
    }

    [global::System.Diagnostics.DebuggerNonUserCodeAttribute]
    public FibReply() {
      OnConstruction();
    }

    partial void OnConstruction();

    [global::System.Diagnostics.DebuggerNonUserCodeAttribute]
    public FibReply(FibReply other) : this() {
      count_ = other.count_;
      _unknownFields = pb::UnknownFieldSet.Clone(other._unknownFields);
    }

    [global::System.Diagnostics.DebuggerNonUserCodeAttribute]
    public FibReply Clone() {
      return new FibReply(this);
    }

    /// <summary>Field number for the "count" field.</summary>
    public const int CountFieldNumber = 1;
    private long count_;
    [global::System.Diagnostics.DebuggerNonUserCodeAttribute]
    public long Count {
      get { return count_; }
      set {
        count_ = value;
      }
    }

    [global::System.Diagnostics.DebuggerNonUserCodeAttribute]
    public override bool Equals(object other) {
      return Equals(other as FibReply);
    }

    [global::System.Diagnostics.DebuggerNonUserCodeAttribute]
    public bool Equals(FibReply other) {
      if (ReferenceEquals(other, null)) {
        return false;
      }
      if (ReferenceEquals(other, this)) {
        return true;
      }
      if (Count != other.Count) return false;
      return Equals(_unknownFields, other._unknownFields);
    }

    [global::System.Diagnostics.DebuggerNonUserCodeAttribute]
    public override int GetHashCode() {
      int hash = 1;
      if (Count != 0L) hash ^= Count.GetHashCode();
      if (_unknownFields != null) {
        hash ^= _unknownFields.GetHashCode();
      }
      return hash;
    }

    [global::System.Diagnostics.DebuggerNonUserCodeAttribute]
    public override string ToString() {
      return pb::JsonFormatter.ToDiagnosticString(this);
    }

    [global::System.Diagnostics.DebuggerNonUserCodeAttribute]
    public void WriteTo(pb::CodedOutputStream output) {
      if (Count != 0L) {
        output.WriteRawTag(8);
        output.WriteInt64(Count);
      }
      if (_unknownFields != null) {
        _unknownFields.WriteTo(output);
      }
    }

    [global::System.Diagnostics.DebuggerNonUserCodeAttribute]
    public int CalculateSize() {
      int size = 0;
      if (Count != 0L) {
        size += 1 + pb::CodedOutputStream.ComputeInt64Size(Count);
      }
      if (_unknownFields != null) {
        size += _unknownFields.CalculateSize();
      }
      return size;
    }

    [global::System.Diagnostics.DebuggerNonUserCodeAttribute]
    public void MergeFrom(FibReply other) {
      if (other == null) {
        return;
      }
      if (other.Count != 0L) {
        Count = other.Count;
      }
      _unknownFields = pb::UnknownFieldSet.MergeFrom(_unknownFields, other._unknownFields);
    }

    [global::System.Diagnostics.DebuggerNonUserCodeAttribute]
    public void MergeFrom(pb::CodedInputStream input) {
      uint tag;
      while ((tag = input.ReadTag()) != 0) {
        switch(tag) {
          default:
            _unknownFields = pb::UnknownFieldSet.MergeFieldFrom(_unknownFields, input);
            break;
          case 8: {
            Count = input.ReadInt64();
            break;
          }
        }
      }
    }

  }

  #endregion
}
#endregion Designer generated code
```


# Kotlin
```kotlin
package liang.cxx.rommel.evaluationbar

import android.content.Context
import android.graphics.*
import android.util.AttributeSet
import android.view.MotionEvent
import android.view.View

/**
 * Created by Rommel on 2017/12/1.
 */

class EvaluationProgressBar : View {
    private var m_density: Float = 0.0f//dp
    private var img_height: Int = 30//滑块和进度条高度(默认三十)
    private var m_content: Context? = null
    //滑块样式
    private var slider: Bitmap? = null
    private var m_laughing: Bitmap? = null
    private var m_satisfactiong: Bitmap? = null
    private var m_sad: Bitmap? = null
    //滑块位置
    var image_postion: Float = 0.0f
    private //滑块宽度
    var image_weight: Float = 0.0f
    private //控件的宽和高
    var m_weight: Float = 0.0f
    private var m_height: Float = 0.0f
    private //每段的长度
    var m_cell: Float = 0.0f
    private //是否为第一次加载
    var is_first = true
    //画笔
    private var m_color_back: Paint? = null
    private var m_color_back_line: Paint? = null
    private var m_color_back_diver_one: Paint? = null
    private var m_color_back_diver_two: Paint? = null
    private var m_color_back_wall: Paint? = null
    private var back_line = "#C7C7C7"
    private var back_sad = "#CECECE"//差评背景
    private var back_satisfactiong = "#FFEAB0"//中评背景
    private var back_satisfactiong_line = "#FFD68C"//中评线框
    private var back_laughing = "#FFBBBB"//好评背景
    private var back_laughing_line = "#FF9F9F"//好评背景
    private var back_baseground = "#F8F8F8"//好评背景
    private var back_satisfactiong_diver = "#FFD68C"//中评分割线
    private var back_laughing_diver = "#FF9F9F"//好评分割线
    //触摸和滑动是否在滑块上
    private var is_touch_scope = false

    //滑动监听事件
    private var volChangeListener: VolChangeListener? = null
    private var vol = 2

    constructor(context: Context) : super(context) {
        init(context)
    }

    constructor(context: Context, attrs: AttributeSet?) : super(context, attrs) {
        init(context)
    }

    constructor(context: Context, attrs: AttributeSet?, defStyleAttr: Int) : super(context, attrs, defStyleAttr) {
        init(context)
    }

    private fun init(context: Context) {
        m_density = context.resources.displayMetrics.density
        m_laughing = zoomImg(BitmapFactory.decodeResource(resources, R.mipmap.icon_laughing))
        m_satisfactiong = zoomImg(BitmapFactory.decodeResource(resources, R.mipmap.icon_satisfaction))
        m_sad = zoomImg(BitmapFactory.decodeResource(resources, R.mipmap.icon_sad))
        image_weight = m_laughing!!.width.toFloat()
        m_content = context
        m_height = (m_laughing!!.height + 1).toFloat()
        //画笔初始化
        m_color_back = Paint()
        m_color_back_line = Paint()
        m_color_back_line!!.style = Paint.Style.STROKE
        m_color_back_line!!.isAntiAlias = true
        m_color_back_line!!.strokeMiter = 1.0f
        m_color_back_diver_one = Paint()
        m_color_back_diver_two = Paint()
        m_color_back_diver_one!!.strokeMiter = 1.0F
        m_color_back_diver_two!!.strokeMiter = 1.0F
        m_color_back_wall = Paint()
        m_color_back_wall!!.color = Color.parseColor(back_line)
        m_color_back_wall!!.style = Paint.Style.STROKE
        m_color_back_wall!!.isAntiAlias = true
        m_color_back_wall!!.strokeWidth = 1.0f
    }

    override fun onDraw(canvas: Canvas) {
        super.onDraw(canvas)
        canvas.drawFilter = PaintFlagsDrawFilter(0, Paint.ANTI_ALIAS_FLAG or Paint.FILTER_BITMAP_FLAG)
        image_postion = if (image_postion < image_weight / 2) image_weight / 2 else image_postion
        image_postion = if (image_postion > m_weight - image_weight / 2) m_weight - image_weight / 2 else image_postion
        if (image_postion <= m_cell) {
            //差评
            slider = m_sad
            m_color_back!!.color = Color.parseColor(back_sad)
            m_color_back_line!!.color = Color.parseColor(back_line)
        } else if (image_postion > m_cell && image_postion <= m_cell * 2) {
            //中评
            slider = m_satisfactiong
            m_color_back!!.color = Color.parseColor(back_satisfactiong)
            m_color_back_line!!.color = Color.parseColor(back_satisfactiong_line)
        } else if (image_postion > m_cell * 2) {
            //好评
            slider = m_laughing
            m_color_back!!.color = Color.parseColor(back_laughing)
            m_color_back_line!!.color = Color.parseColor(back_laughing_line)
        }
        drawWall(canvas)
        drawBckground(canvas)
        drawDiverLine(canvas)
        drawImage(canvas)
    }

    //绘制最外层线和整体背景
    fun drawWall(canvas: Canvas) {
        //外边框
        var rectf = RectF(2f, 0f, m_weight, m_height)
        canvas.drawRoundRect(rectf, m_height / 2, m_height / 2, m_color_back_wall)
        //基础背景
        var back_base = RectF(3f, 1f, m_weight - 1, m_height - 1)
        var back_base_color = Paint()
        back_base_color.color = Color.parseColor(back_baseground)
        canvas.drawRoundRect(back_base, m_height / 2, m_height / 2, back_base_color)
    }

    //绘制进度颜色
    fun drawBckground(canvas: Canvas) {
        var rectf = RectF(2f, 1f, image_postion + image_weight / 2 - 1, m_height - 1)
        canvas.drawRoundRect(rectf, m_height / 2, m_height / 2, m_color_back)
    }

    //根据滑块位置改变边框和分割线颜色
    fun drawDiverLine(canvas: Canvas) {
        //根据滑块位置改变外层边框颜色
        var rectF = RectF(2f, 0f, image_postion + image_weight / 2, m_height)
        canvas.drawRoundRect(rectF, m_height / 2, m_height / 2, m_color_back_line)
        if (image_postion <= m_cell) {
            //差评
            m_color_back_diver_one!!.color = Color.parseColor(back_line)
            canvas.drawLine(m_weight / 3, 0f, m_weight / 3, m_height, m_color_back_diver_one)
            m_color_back_diver_two!!.color = Color.parseColor(back_line)
            canvas.drawLine(m_weight * 2 / 3, 0f, m_weight * 2 / 3, m_height, m_color_back_diver_two)
        } else if (image_postion > m_cell && image_postion <= m_cell * 2) {
            //中评
            m_color_back_diver_one!!.color = Color.parseColor(back_satisfactiong_diver)
            canvas.drawLine(m_weight / 3, 0f, m_weight / 3, m_height, m_color_back_diver_one)
            m_color_back_diver_two!!.color = Color.parseColor(back_line)
            canvas.drawLine(m_weight * 2 / 3, 0f, m_weight * 2 / 3, m_height, m_color_back_diver_two)
        } else if (image_postion > m_cell * 2) {
            //好评
            m_color_back_diver_one!!.color = Color.parseColor(back_laughing_diver)
            canvas.drawLine(m_weight / 3, 0f, m_weight / 3, m_height, m_color_back_diver_one)
            m_color_back_diver_two!!.color = Color.parseColor(back_laughing_diver)
            canvas.drawLine(m_weight * 2 / 3, 0f, m_weight * 2 / 3, m_height, m_color_back_diver_two)
        }
    }

    //绘制滑块
    fun drawImage(canvas: Canvas) {
        canvas.drawBitmap(slider, image_postion - image_weight / 2 - 2, 1f, Paint())
    }

    override fun onLayout(changed: Boolean, left: Int, top: Int, right: Int, bottom: Int) {
        super.onLayout(changed, left, top, right, bottom)
        m_weight = (width - 2).toFloat()
        m_cell = m_weight / 3
        if (is_first) {
            //首次加载默认值
            image_postion = m_weight * 2 / 3
            is_first = false
        }

    }

    override fun onMeasure(widthMeasureSpec: Int, heightMeasureSpec: Int) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec)
        var width = getSize((200 * m_density).toInt(), widthMeasureSpec)
        var height = getSize(m_laughing!!.width + 1, heightMeasureSpec)
        setMeasuredDimension(width, height)
    }

    private fun getSize(defaultSize: Int, measureSpec: Int): Int {
        var m_size = defaultSize
        var mode = View.MeasureSpec.getMode(measureSpec)
        var size = MeasureSpec.getSize(measureSpec)
        when (mode) {
            MeasureSpec.AT_MOST -> {
                //当前尺寸是当前View能取的最大尺寸-wrap_content
                m_size = defaultSize
            }
            MeasureSpec.EXACTLY -> {
                //当前的尺寸就是当前View应该取的尺寸-match_parent
                m_size = size
            }
            MeasureSpec.UNSPECIFIED -> {
                //父容器没有对当前View有任何限制，当前View可以任意取尺寸-固定尺寸（如100dp
                m_size = size
            }
        }
        return m_size
    }

    override fun onTouchEvent(event: MotionEvent): Boolean {
        var x_postion = event.x
        when (event.action) {
            MotionEvent.ACTION_DOWN -> {
                if (x_postion > image_postion - image_weight / 2
                        && x < image_postion + image_weight / 2) {
                    //触摸事件在滑块上
                    image_postion = x_postion
                    is_touch_scope = true
                } else {
                    //触摸事件不在滑块上
                    image_postion = x_postion
                    setImgPostion()
                    invalidate()
                    onChange()
                    return false
                }
            }
            MotionEvent.ACTION_MOVE -> {
                if (is_touch_scope) {
                    image_postion = x_postion
                    invalidate()
                    onChange()
                }
            }
            MotionEvent.ACTION_UP -> {
                if (is_touch_scope) {
                    is_touch_scope = false
                    setImgPostion()
                }
            }
            MotionEvent.ACTION_CANCEL -> {
                if (is_touch_scope) {
                    is_touch_scope = false
                    setImgPostion()
                }
            }
        }
        return true
    }

    private fun setImgPostion() {
        //松手后滑块走到指定的位置
        if (image_postion <= m_cell) {
            //差评
            image_postion = m_cell
        } else if (image_postion > m_cell && image_postion <= m_cell * 2) {
            //中评
            image_postion = m_cell * 2
        } else if (image_postion > m_cell * 2) {
            //好评
            image_postion = m_cell * 3
        }
        invalidate()
        onChange()
    }

    //滑块位置变化监听
    fun onChange() {
        if (volChangeListener != null) {
            if (image_postion <= m_cell) {
                vol = 1
            } else if (image_postion > m_cell && image_postion <= 2 * m_cell) {
                vol = 2
            } else if (image_postion > 2 * m_cell) {
                vol = 3
            }
            volChangeListener!!.onVolChange(vol)
        }
    }

    private fun zoomImg(btm: Bitmap): Bitmap {
        //获取图片宽高
        var width = btm.width
        var height = btm.height
        //计算缩放比例
        var scaleWidth = (img_height * m_density) / width
        var scaleHeight = (img_height * m_density) / height
        //获取Matrix参数
        var matrix = Matrix()
        matrix.postScale(scaleWidth, scaleHeight)
        //返回新的图片
        return Bitmap.createBitmap(btm, 0, 0, width, height, matrix, true)
    }

    interface VolChangeListener {
        fun onVolChange(vol: Int)
    }

    fun volVhangeListener(volChangeListener :VolChangeListener) {
        this.volChangeListener = volChangeListener
    }
}
```


# HTML
```html
<xmp>
<!DOCTYPE html>
<html lang="zh">
<head>
    <meta charset="UTF-8">
	<meta name="viewport" content="width=device-width, 
                                 initial-scale=1.0, 
                                 maximum-scale=1.0, 
                                 user-scalable=no">
	
	<link rel="stylesheet" href="http://cdn.staticfile.org/twitter-bootstrap/3.3.6/css/bootstrap.min.css">
	<link rel="stylesheet" href="http://cdn.staticfile.org/twitter-bootstrap/3.3.6/css/bootstrap-theme.min.css">
	<script src="http://cdn.staticfile.org/jquery/2.1.4/jquery.min.js"></script>
	<script src="http://cdn.staticfile.org/twitter-bootstrap/3.3.6/js/bootstrap.min.js"></script>

	<title>测试一下bootstrap</title>
</head>
<body>
<div class="container">
<ol class="breadcrumb">
  <li><a href="#">主页</a></li>
  <li><a href="#">新闻资讯</a></li>
  <li class="active">新闻正文</li>
</ol>
<div class="page-header">
  <h1>这是新闻的标题
  <br><small></small></h1>
</div>
	<div class="sstarter-template">
		<p class="lead">Use this dow project.<br> All yoy barebones HTML document.</p>
		<p>Google（中文名：谷歌），是一家美国的跨国科技企业，致力于互联网搜索、云计算、广告技术等领域，开发并提供大量基于互联网的产品与服务，其主要利润来自于AdWords等广告服务。Google由当时在斯坦福大学攻读理工博士的拉里·佩奇和谢尔盖·布卢姆共同创建，因此两人也被称为“Google Guys”。</p>
		<p class="lead">这一段的内容<strong>突出</strong>显示看看效果</p>
		<address>
  <strong>Twitter, Inc.</strong><br>
  795 Folsom Ave, Suite 600<br>
  San Francisco, CA 94107<br>
  <abbr title="Phone">P:</abbr> (123) 456-7890
</address>

<table class="table table-striped">
      <thead>
        <tr>
          <th>#</th>
          <th>First Name</th>
          <th>Last Name</th>
          <th>Username</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th scope="row">1</th>
          <td>Mark</td>
          <td>Otto</td>
          <td>@mdo</td>
        </tr>
        <tr>
          <th scope="row">2</th>
          <td>Jacob</td>
          <td>Thornton</td>
          <td>@fat</td>
        </tr>
        <tr>
          <th scope="row">3</th>
          <td>Larry</td>
          <td>the Bird</td>
          <td>@twitter</td>
        </tr>
      </tbody>
    </table>
	</div>
	<div class="progress">
  <div class="progress-bar progress-bar-danger progress-bar-striped active" role="progressbar" aria-valuenow="45" aria-valuemin="0" aria-valuemax="100" style="width: 80%">
    <span class="sr-only">45% Complete</span>
  </div>
</div>
	<nav>
  <ul class="pager">
    <li><a href="#"><span class="glyphicon glyphicon-chevron-left" aria-hidden="true"></span>前一篇新闻</a></li>
    <li><a href="#">后一篇新闻<span class="glyphicon glyphicon-chevron-right" aria-hidden="true"></span></a></li>
  </ul>
</nav>
	<button class="btn btn-danger btn-block btn-lg btn-primary" type="button">退出登录</button>
</div>
</body>
</html>
<xmp>
```

# CSS
```css
.top{background-color: gray;
    font-size: 14px; 
      
}
.top p{display: inline-block;
 line-height: 20px;
 color:white;
 margin: auto;
 overflow: hidden;
}
.top P:nth-child(1){margin-left: 10%;
}
.top P:nth-child(2){margin-left: 50%;
}
.top P:nth-child(3){margin-left: 5%;
}

.header img{
  float: left;
  margin-left: 8%;	
}

.nav-tabs {
  border-bottom: 0px solid #ddd;
  height: 82px;
  margin-top: 20px;
  
 
}
.nav-tabs > li {
  float: left;
  margin-top: 20px;
  font-size: 18px;
  margin-left: 20px;
  
  
  
}
.nav-tabs > li > a {
  margin-right: 30px;
  line-height: 1.42857143;
  border: 0px solid transparent;
  border-radius: 4px 4px 0 0;
}
.nav-tabs > li > a:hover {
  border-color: #eee #eee #ddd;
}
.nav-tabs > li.active > a,
.nav-tabs > li.active > a:hover,
.nav-tabs > li.active > a:focus {
  color: #555;
  cursor: default;
  background-color: #fff;
  border: 1px solid #ddd;
  border-bottom-color: transparent;
}


.banner img{
	width: 100%;
}

.xiaotu{text-align: center;
        padding-left: 50px; 
        float: left;		
}

.survey{background-color: gray;
       font-size: 14px;
       height: 60px;
       position: relative;
      
      
}
.survey p{display:inline-block;
 color: white;
 margin: 20px auto;
 overflow: hidden;

}
.survey P:nth-child(1){margin-left: 10%;
}
.survey P:nth-child(2){margin-left: 1%;
}


#nav{padding: 0;
    margin-left: 40%;
    position:absolute;
    display: inline-block;
}
#nav ul{list-style-type: none; }
#nav ul li{float: left;}
#nav ul li a{text-align: center; 
padding:20px; 
display:block; 
text-decoration:none; 
color:white;
line-height: 20px; 
}
#nav ul li a:hover{
	background-color:white;
    color:#31B0D5;
}
.Introduced{text-align: center;
           margin-top: 50px;}

.Introduced p{
   max-width: 900px;
   margin: 30px auto;	
   text-align: left;
}


```

# Javascript
```javascript
'use strict';

var defaults = require('../core/core.defaults');
var Element = require('../core/core.element');
var helpers = require('../helpers/index');
var layouts = require('../core/core.layouts');

var noop = helpers.noop;

defaults._set('global', {
	title: {
		display: false,
		fontStyle: 'bold',
		fullWidth: true,
		lineHeight: 1.2,
		padding: 10,
		position: 'top',
		text: '',
		weight: 2000         // by default greater than legend (1000) to be above
	}
});

/**
 * IMPORTANT: this class is exposed publicly as Chart.Legend, backward compatibility required!
 */
var Title = Element.extend({
	initialize: function(config) {
		var me = this;
		helpers.extend(me, config);

		// Contains hit boxes for each dataset (in dataset order)
		me.legendHitBoxes = [];
	},

	// These methods are ordered by lifecycle. Utilities then follow.

	beforeUpdate: noop,
	update: function(maxWidth, maxHeight, margins) {
		var me = this;

		// Update Lifecycle - Probably don't want to ever extend or overwrite this function ;)
		me.beforeUpdate();

		// Absorb the master measurements
		me.maxWidth = maxWidth;
		me.maxHeight = maxHeight;
		me.margins = margins;

		// Dimensions
		me.beforeSetDimensions();
		me.setDimensions();
		me.afterSetDimensions();
		// Labels
		me.beforeBuildLabels();
		me.buildLabels();
		me.afterBuildLabels();

		// Fit
		me.beforeFit();
		me.fit();
		me.afterFit();
		//
		me.afterUpdate();
		return me.minSize;

	},
	afterUpdate: noop,

	//

	beforeSetDimensions: noop,
	setDimensions: function() {
		var me = this;
		// Set the unconstrained dimension before label rotation
		if (me.isHorizontal()) {
			// Reset position before calculating rotation
			me.width = me.maxWidth;
			me.left = 0;
			me.right = me.width;
		} else {
			me.height = me.maxHeight;

			// Reset position before calculating rotation
			me.top = 0;
			me.bottom = me.height;
		}

		// Reset padding
		me.paddingLeft = 0;
		me.paddingTop = 0;
		me.paddingRight = 0;
		me.paddingBottom = 0;

		// Reset minSize
		me.minSize = {
			width: 0,
			height: 0
		};
	},
	afterSetDimensions: noop,

	//

	beforeBuildLabels: noop,
	buildLabels: noop,
	afterBuildLabels: noop,

	//

	beforeFit: noop,
	fit: function() {
		var me = this;
		var valueOrDefault = helpers.valueOrDefault;
		var opts = me.options;
		var display = opts.display;
		var fontSize = valueOrDefault(opts.fontSize, defaults.global.defaultFontSize);
		var minSize = me.minSize;
		var lineCount = helpers.isArray(opts.text) ? opts.text.length : 1;
		var lineHeight = helpers.options.toLineHeight(opts.lineHeight, fontSize);
		var textSize = display ? (lineCount * lineHeight) + (opts.padding * 2) : 0;

		if (me.isHorizontal()) {
			minSize.width = me.maxWidth; // fill all the width
			minSize.height = textSize;
		} else {
			minSize.width = textSize;
			minSize.height = me.maxHeight; // fill all the height
		}

		me.width = minSize.width;
		me.height = minSize.height;

	},
	afterFit: noop,

	// Shared Methods
	isHorizontal: function() {
		var pos = this.options.position;
		return pos === 'top' || pos === 'bottom';
	},

	// Actually draw the title block on the canvas
	draw: function() {
		var me = this;
		var ctx = me.ctx;
		var valueOrDefault = helpers.valueOrDefault;
		var opts = me.options;
		var globalDefaults = defaults.global;

		if (opts.display) {
			var fontSize = valueOrDefault(opts.fontSize, globalDefaults.defaultFontSize);
			var fontStyle = valueOrDefault(opts.fontStyle, globalDefaults.defaultFontStyle);
			var fontFamily = valueOrDefault(opts.fontFamily, globalDefaults.defaultFontFamily);
			var titleFont = helpers.fontString(fontSize, fontStyle, fontFamily);
			var lineHeight = helpers.options.toLineHeight(opts.lineHeight, fontSize);
			var offset = lineHeight / 2 + opts.padding;
			var rotation = 0;
			var top = me.top;
			var left = me.left;
			var bottom = me.bottom;
			var right = me.right;
			var maxWidth, titleX, titleY;

			ctx.fillStyle = valueOrDefault(opts.fontColor, globalDefaults.defaultFontColor); // render in correct colour
			ctx.font = titleFont;

			// Horizontal
			if (me.isHorizontal()) {
				titleX = left + ((right - left) / 2); // midpoint of the width
				titleY = top + offset;
				maxWidth = right - left;
			} else {
				titleX = opts.position === 'left' ? left + offset : right - offset;
				titleY = top + ((bottom - top) / 2);
				maxWidth = bottom - top;
				rotation = Math.PI * (opts.position === 'left' ? -0.5 : 0.5);
			}

			ctx.save();
			ctx.translate(titleX, titleY);
			ctx.rotate(rotation);
			ctx.textAlign = 'center';
			ctx.textBaseline = 'middle';

			var text = opts.text;
			if (helpers.isArray(text)) {
				var y = 0;
				for (var i = 0; i < text.length; ++i) {
					ctx.fillText(text[i], 0, y, maxWidth);
					y += lineHeight;
				}
			} else {
				ctx.fillText(text, 0, 0, maxWidth);
			}

			ctx.restore();
		}
	}
});

function createNewTitleBlockAndAttach(chart, titleOpts) {
	var title = new Title({
		ctx: chart.ctx,
		options: titleOpts,
		chart: chart
	});

	layouts.configure(chart, title, titleOpts);
	layouts.addBox(chart, title);
	chart.titleBlock = title;
}

module.exports = {
	id: 'title',

	/**
	 * Backward compatibility: since 2.1.5, the title is registered as a plugin, making
	 * Chart.Title obsolete. To avoid a breaking change, we export the Title as part of
	 * the plugin, which one will be re-exposed in the chart.js file.
	 * https://github.com/chartjs/Chart.js/pull/2640
	 * @private
	 */
	_element: Title,

	beforeInit: function(chart) {
		var titleOpts = chart.options.title;

		if (titleOpts) {
			createNewTitleBlockAndAttach(chart, titleOpts);
		}
	},

	beforeUpdate: function(chart) {
		var titleOpts = chart.options.title;
		var titleBlock = chart.titleBlock;

		if (titleOpts) {
			helpers.mergeIf(titleOpts, defaults.global.title);

			if (titleBlock) {
				layouts.configure(chart, titleBlock, titleOpts);
				titleBlock.options = titleOpts;
			} else {
				createNewTitleBlockAndAttach(chart, titleOpts);
			}
		} else if (titleBlock) {
			layouts.removeBox(chart, titleBlock);
			delete chart.titleBlock;
		}
	}
};
```


# JSON

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "(gdb) Launch", 
            "type": "cppdbg",
            "request": "launch", 
            "program": "${fileDirname}/${fileBasenameNoExtension}.exe", 
            "stopAtEntry": false, 
            "cwd": "${workspaceRoot}",
            "externalConsole": true, 
            "linux": {
                "MIMode": "gdb"
            },
            "osx": {
                "MIMode": "lldb"
            },
            "windows": { 
                "MIMode": "gdb",
                "miDebuggerPath": "gdb.exe"
            }
        }
    ]
}

```

# SQL

```sql
 SELECT Company, OrderNumber FROM Orders ORDER BY Company DESC, OrderNumber ASC
 update whut_student SET grade=2017 where id>1003740
 update whut_student SET id=id+1 where id>0
 select COUNT(student_id) from whut_student

 select * from whut_student
 where student_id in (select  student_id  from  whut_student  group  by  student_id  having  count(student_id) > 1)

 delete from whut_student where id<=137
 UPDATE whut_student SET DEGREE="硕士" WHERE id>0;

 SELECT * from whut_student where student_id=1049731702891;

 DELETE FROM whut_student WHERE id=1008281

 SELECT COUNT(*)  num,student_id FROM whut_student GROUP BY student_id ORDER BY num DESC ;
```

# Properties

```ini
## 开发/测试/生产环境分别对应dev/test/prod

server.port=1000
debug=true
trace=false
spring.application.name=blog-web-server
spring.profiles.active=dev

spring.datasource.name=datasource
spring.datasource.type=com.alibaba.druid.pool.DruidDataSource
spring.datasource.druid.filters=stat
spring.datasource.druid.driver-class-name=com.mysql.jdbc.Driver
spring.datasource.druid.url=jdbc:mysql://127.0.0.1:3306/acanx?useUnicode=true&characterEncoding=UTF-8&allowMultiQueries=true
spring.datasource.druid.username=root
spring.datasource.druid.password=root
spring.datasource.druid.initial-size=1
spring.datasource.druid.min-idle=1
spring.datasource.druid.max-active=20
spring.datasource.druid.max-wait=60000
spring.datasource.druid.time-between-eviction-runs-millis=60000
spring.datasource.druid.min-evictable-idle-time-millis=300000
spring.datasource.druid.validation-query=SELECT 'x'
spring.datasource.druid.test-while-idle=true
spring.datasource.druid.test-on-borrow=false
spring.datasource.druid.test-on-return=false
spring.datasource.druid.pool-prepared-statements=false
spring.datasource.druid.max-pool-prepared-statement-per-connection-size=20
```


# yml
```yml
# 设置集群的名字，对应为http://127.0.0.1:9200/中的"cluster_name" : "acanx-es-demo",
cluster.name: acanx-es-demo
# 节点名字，对应为http://127.0.0.1:9200/中的"name" : "node-1001",
node.name: node-1001
node.master: true
node.data: true
# 修改ES的监听地址，以便其他机器也可以访问
network.host: 0.0.0.0
# 默认端口
http.port: 9200
# 增加新的CORS参数，这样head插件可以访问ES
http.cors.enabled: true
http.cors.allow-origin: "*"
```


# Bash
```bash
#!/bin/bash
# - main program
set -o errexit -o noglob -o pipefail

# Configuration constants:
# - Use upper case, underscore-separated names
# - Use `readonly` keyword to highlight it is a constant
readonly GREETING_FORMAT="Hello %s!\n"

# Global variables:

function sayHello {
    # Variable declarations:
    # - Always declared using `local` keyword
    # - Function parameters declared at the beginning of function
    # - Use meaningful names
    # - Use all lower case, underscrore-separated names
    local greeted_name="$1"

    # Variable references:
    # - Are always wrapped in ${ }
    # - Are properly quoted to avoid issues with embedded whitespace
    if [ -z "${greeted_name}" ]; then
        # Return status:
        # - Use a non-zero value if something went wrong
        # - If none is specified - zero (success) is used by default
        return 1
    else
        if [ "${greeted_name}" == '-' ]; then
            read greeted_name
        fi
        printf "${GREETING_FORMAT}" "${greeted_name}"
    fi
}

function toUpperCase {
    tr "[a-z]" "[A-Z]"
}

function printError {
    local message="$1"
    local red_color='\e[31;1m'
    local no_color='\e[m'

    printf '\n'${red_color}'ERROR:'${no_color}' %s\n' "${message}" 1>&2
}

# Main program

sayHello "Luke Skywalker"

echo "Han Solo" | sayHello -

echo "Master Yoda" | sayHello - | toUpperCase

if ! sayHello; then
    printError "You gave no name!"
    exit 1
fi

```


# Shell
```javascript
#!/bin/sh
echo '============卡密管理==============='
echo '创建卡密类型如下:'
arr=`redis-cli -h 172.17.95.112 keys hardware:bus:*`
 for i in ${arr[@]};
        do
        bus=`echo $i | awk -F ':' '{print $3}'`
        value=`redis-cli --raw -h 172.17.95.112 get $i | grep -Po '"name":".*?"' | awk -F ':' '{print $2}'`
         echo $bus、$value ;
 done

read -p '请输入操作:' typevalue
if [ "$typevalue" == "exit" ];then
        exit
fi

isexist=`redis-cli --raw -h 172.17.95.112 get "hardware:bus:$typevalue:"`
if [ "$isexist" == "" ];then
        echo '输入类型不存在'
        sh $0
fi


read -p '生成数量:' createNum

echo $createNum

echo "====================卡密=========================编号:$typevalue"
outFile="$typevalue-$RANDOM.log"
for((i=0;i<$createNum;i++));
do
        card=`date +%s%N | md5sum | head -c 8`
        redis-cli -h 172.17.95.112 set "hardware:card:$card:" "$typevalue" >/dev/null
        echo "$card">>$outFile
        echo "$card"
done
echo "输出文件路径为`pwd`/$outFile"
echo '====================卡密========================='
exit
```


# PowerShell
```powershell
title 修改shift+右击在此处打开PowerShell
mode con cols=73 lines=20
rem 验证是否是以管理员身份运行
set TempFile_Name=%SystemRoot%\System32\BatTestUACin_SysRt%Random%.batemp
( echo "BAT Test UAC in Temp" >%TempFile_Name% ) 1>nul 2>nul
if not exist %TempFile_Name% (
color 0c
echo.
echo.
echo 错误：没有以管理员身份运行该工具。
echo 该工具将无法使用！
echo.
echo 方法：
echo 找到该工具，右击"以管理员身份运行"。
echo.
echo 按任意键退出该工具..... & pause >nul
exit
)
echo 初始化中，请稍后.....
rem 设置"HKCR\Directory\Background\shell"的所有者为:Everyone
>>fcc_owner.inf echo.[Version]
>>fcc_owner.inf echo.Signature = "$Chicago$"
>>fcc_owner.inf echo.
>>fcc_owner.inf echo.[Registry Keys]
>>fcc_owner.inf echo."CLASSES_ROOT\Directory\Background\shell", 0, "O:WD"
secedit /configure /db fcc_owner.sdb /cfg fcc_owner.inf /log fcc_owner.log
del fcc_owner.*
:main
cls
color 0a
echo *************************************************************************
echo.
echo 该工具用于更改新版windows10系统shift+右键以何种方式打开命令窗口。
echo 系统默认“在此处打开PowerShell”，
echo 此工具可以在“PowerShell”和“CMD”之间切换。
echo.
echo ★注意！请使用管理员权限运行改工具！
echo.
echo 请选择操作：
echo 1、修改为“在此处打开CMD窗口”
echo 2、修改为“在此处打开PowerShell窗口”
echo 3、同时显示1和2
echo.
echo by:frogchou
echo website:frogchou.com
echo *************************************************************************
echo.
set /p var=请输入序号并回车开始修改：
if "%var%"=="1" goto changeCMD
if "%var%"=="2" goto changePowerShell
if "%var%"=="3" (goto changeShowAll) else (goto error)
:changeCMD
reg delete HKCR\Directory\Background\shell\cmd /v HideBasedOnVelocityId /f
reg add HKCR\Directory\Background\shell\cmd /v ShowBasedOnVelocityId /t REG_DWORD /d 6527944 /f
reg delete HKCR\Directory\Background\shell\Powershell /v ShowBasedOnVelocityId /f
reg add HKCR\Directory\Background\shell\Powershell /v HideBasedOnVelocityId /t REG_DWORD /d 6527944 /f
goto end
:changePowerShell
reg delete HKCR\Directory\Background\shell\cmd /v ShowBasedOnVelocityId /f
reg add HKCR\Directory\Background\shell\cmd /v HideBasedOnVelocityId /t REG_DWORD /d 6527944 /f
reg delete HKCR\Directory\Background\shell\Powershell /v HideBasedOnVelocityId /f
reg add HKCR\Directory\Background\shell\Powershell /v ShowBasedOnVelocityId /t REG_DWORD /d 6527944 /f
goto end
:changeShowAll
reg delete HKCR\Directory\Background\shell\cmd /v HideBasedOnVelocityId /f
reg add HKCR\Directory\Background\shell\cmd /v ShowBasedOnVelocityId /t REG_DWORD /d 6527944 /f
reg delete HKCR\Directory\Background\shell\Powershell /v HideBasedOnVelocityId /f
reg add HKCR\Directory\Background\shell\Powershell /v ShowBasedOnVelocityId /t REG_DWORD /d 6527944 /f
goto end
:end
echo.
echo 修改完成！按任意键退出.
pause >nul
exit
:error
echo.
echo 选择错误！按任意键返回主菜单.
pause >nul
goto main
```


# CMD
```cmd
@echo off 
echo 定时关机DOS命令，使用方式: "sdat 23:12:12"; 
echo sdat 为命令名称，23:12:12为当天的时间参数
::set yy=%date:~0,4%
::set mm=%date:~5,2%
::set dd=%date:~8,2%
set h=%time:~0,2%
set m=%time:~3,2%
set s=%time:~6,2%
::echo %yy% %mm% %dd% %h% %m% %s%
::echo %1 %2 %3
set /a ts=(%1 - %h%)*3600 + (%2 - %m%)*60 + (%3 - %s%)
IF /i %ts% LSS 1 (
echo ERROR:Shutdown time must be time in future;
) ELSE (
echo INFO:System will shutdown after %ts% seconds at %1 : %2 ：%3; 
shutdown /s /t %ts%
)
::pause 
```



# Bat
```bat
@echo off
echo.
echo MySQL数据库备份
set del_date_back=15
set all_data_back_value=1
set haoya_bin_path="C:\Program Files\HaoZip\"
set mysql_bin_path="C:\wamp\bin\mysql\mysql5.5.24\bin\"
set mysql_data_path="C:\wamp\bin\mysql\mysql5.5.24\data\"
::set apache_bin_path="C:\wamp\bin\apache\apache2.2.22\bin\"
set apache_data_path="C:\wamp\"
set data_back_path="C:\dtsbc_data_back\"
set del_file_path="C:\wamp\www\dtsbc\Admin\Runtime\"
set Ymd="%date:~,4%%date:~5,2%%date:~8,2%"
set /a t=100+%time:~,2%
set bakFileNameTail=_BAK_%date:~,4%%date:~5,2%%date:~8,2%_%t:~1%%time:~3,2%%time:~6,2%
echo *****************************
echo.
echo 今天是： %date%_%time%
echo 好压执行路径为： %haoya_bin_path%
echo mysql执行路径为： %mysql_bin_path%
echo mysql数据路径为： %mysql_data_path%
::echo apache执行路径为： %apache_bin_path%
echo apache数据路径为： %apache_data_path%
echo.
echo *****************************
:: HaoZipC a -tzip 20161221.zip config\*
::set mysql_bin_path = "C:\wamp\bin\mysql\mysql5.5.24\bin"
::set apache_bin_path = "C:\wamp\bin\apache\apache2.2.22\bin"
::set haoya_bin_path = "C:\Program Files\HaoZip"
::创建备份的数据的目录

md %data_back_path%%Ymd%
%mysql_bin_path%\mysqldump -u root --default-character-set=utf8 --databases dtsbc > %data_back_path%%Ymd%\dtsbc_%bakFileNameTail%.sql
%mysql_bin_path%\mysqldump -u root --default-character-set=utf8 --databases mysql > %data_back_path%%Ymd%\mysql_%bakFileNameTail%.sql
%mysql_bin_path%\mysqldump -u root --default-character-set=utf8 --databases bbsdata > %data_back_path%%Ymd%\bbsdata_%bakFileNameTail%.sql
net stop wampapache
net stop wampmysqld
rd/s/q %del_file_path%

%haoya_bin_path%HaoZipC a -tzip %data_back_path%%Ymd%\dtsbc_mysql_%Ymd%.zip %mysql_data_path%dtsbc\*
%haoya_bin_path%HaoZipC a -tzip %data_back_path%%Ymd%\bbs_mysql_%Ymd%.zip %mysql_data_path%bbs\*
%haoya_bin_path%HaoZipC a -tzip %data_back_path%%Ymd%\mysql_mysql_%Ymd%.zip %mysql_data_path%mysql\*
%haoya_bin_path%HaoZipC a -tzip %data_back_path%%Ymd%\dtsbc_php_%Ymd%.zip %apache_data_path%dtsbc\*
%haoya_bin_path%HaoZipC a -tzip %data_back_path%%Ymd%\bbs_php_%Ymd%.zip %apache_data_path%bbs\*

if  %date:~9,1% equ %all_data_back_value% ( %haoya_bin_path%HaoZipC a -tzip %data_back_path%%Ymd%\wamp_%Ymd%.zip %apache_data_path%*)
::cmd
net start wampapache
net start wampmysqld
::set/a n=5
set/a y=%date:~,4%,m=1%date:~5,2%-100,d=1%date:~8,2%-100
set/a d-=%del_date_back%
if %d% gtr 0 goto :ok
:ov
set/a "md=31-!(m-5)-!(m-7)-!(m-10)-!(m-12)-!(m-3)*(3-!(y&3))"
set/a d+=md,m-=1
if %m% equ 0 set/a m=12,y-=1
if %d% leq 0 goto :ov
:ok
set/a md=m*100+d+10000
set back_data_value="%y%%md:~1%"
echo %del_date_back%天前的日期为：%back_data_value%
cd %data_back_path%

set /a data_back_value_int=%back_data_value%+0
echo %data_back_value_int%
::单条语句执行的时候需要的是一个% 在批处理中的话，需要两个%%
for /F %%i in ('dir  %data_back_path% /a:d /b') do (if %%i lss %data_back_value_int% ( rd/s/q %%i ))
::for /F %%i in ('dir  %data_back_path% /a:d /b') do (  if %%i lss %data_back_value_int% ( echo %%i ) else ( echo "google" )  )
echo.:P

echo MySQL数据库备份完成，请进行检查。。。

echo.:P
echo.
//pause
::pause
```



# Groovy
```groovy
errpattern = ~/Failures: [1-9]+/
errpatternSum = ~/passed:\d+,.*failed:\d+,/
def ratio
manager.build.logFile.eachLine{ line ->
    errmatcher=errpattern.matcher(line)
    errmatcherSum=errpatternSum.matcher(line)
    if (errmatcher.find()) {
        manager.build.@result = hudson.model.Result.UNSTABLE
    }
    if(errmatcherSum.find()){
        numArray = line.findAll(/\d+/)*.toInteger()
        manager.listener.logger.println(">>> number =  " + numArray)
        ratio = (numArray[1] / numArray[3]) * 100
        manager.listener.logger.println(">>> ratio =  " + ratio + "%")
    }
}

ratio = Math.round(ratio)
//add text next to the job build history
manager.addShortText("failure rate -> " + ratio + "%")
```


# Gradle
```gradle
apply plugin: 'com.android.application'

android {
    compileSdkVersion 25
    buildToolsVersion "25.0.1"
    defaultConfig {
        applicationId "com.grgbanking.school"
        minSdkVersion 15
        targetSdkVersion 25
        versionCode 1
        versionName "1.0"
        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
    dataBinding {
        enabled = true
    }
}

dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    androidTestCompile('com.android.support.test.espresso:espresso-core:2.2.2', {
        exclude group: 'com.android.support', module: 'support-annotations'
    })
    compile 'com.android.support:appcompat-v7:25.0.1'
    compile 'com.jakewharton.rxbinding:rxbinding:0.4.0'

    /*compile 'com.squareup.retrofit2:retrofit:2.1.0'
    compile 'com.squareup.retrofit2:adapter-rxjava:2.0.0-beta4'*/
    compile 'com.squareup.retrofit2:retrofit:2.0.0-beta4'
    compile 'com.squareup.retrofit2:converter-gson:2.0.0-beta4'
    compile 'com.squareup.retrofit2:adapter-rxjava:2.0.0-beta4'

    compile 'com.squareup.okhttp:okhttp:2.7.5'
    compile 'com.alibaba:fastjson:1.2.20'
    compile 'org.apache.commons:commons-lang3:3.5'

    testCompile 'junit:junit:4.12'
}

```

#  Git
```git
# 1.初始化git
git init 

# 2.查看git修改状态：
git status

# 3.将项目保存到仓库
先 git add .
再 git commit -m "备注"

# 4. 查看最新三个仓库中的项目
git log

# 5. 查看所有仓库中的项目
git reflog

# 6.恢复成某个项目
git reset --hard 这里写id

# 7.将github中的项目拷贝下来
git clone 地址

# 8.将本地项目传到github 
git push origin master


# 9.更新本地项目，与gitbub一致
git pull

# 10.将本地项目和新建的远程仓库关联起来，必须是未初始化仓库
git remote add origin git@github.com:michaelliao/learngit.git

# 11.把本地库的内容推送到远程，用git push命令
git push -u origin master    # （由于远程库是空的，我们第一次推送master分支时，加上了-u参数，
				Git不但会把本地的master分支内容推送的远程新的master分支，
				还会把本地的master分支和远程的master分支关联起来，
				在以后的推送或者拉取时就可以简化命令。）

```


···
···
···

# 待添加
```js
connect: {
    server: {
        options: {
            port: 9100,
            hostname: '*',
            base: '.',
            keepalive: true
        }
    }
}
```
# 待添加

```js
this.base_uri = this.config.base_uri || this.prefs.get("app-base_uri") || "http://localhost:9200";
```

# 待添加

```js
this.base_uri = this.config.base_uri || this.prefs.get("app-base_uri") || "http://127.0.0.1:9200";
```


# 待添加
 
```
 npm install
```

# 待添加


# 待添加



