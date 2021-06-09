---
title: PE结构学习
tags:
  - PE文件
  - 汇编
  - 逆向
id: '8'
categories:
  - - 计算机科学
date: 2018-02-21 01:12:00
---

参考资料：  
滴水逆向2015-03-16期。  
1、PE结构简介  
本论坛PE的解析的帖子以及教程非常多，我这里只是介绍一下学习的过程与思路，具体的内容还是需要自己去记忆与理解。结合PEtool或者其他的PE解析工具以及下载链接中的PDF文件，首先将pe的每个字段所在位置，整体分布，由一个宏观的把握。PE字段的命名有一定的规律，了解了文件偏移，虚拟地址，相对虚拟地址这些概念之后，一部分字段就可以猜测出含义，部分没有规律的，可以百度。  
PE文件总览 [https://pan.baidu.com/s/1oAkP0TG](https://pan.baidu.com/s/1oAkP0TG) 密码: smxa
<!-- more -->
  
2、滴水逆向2015-03-16期课后做业  
模拟一个PE文件在内存中的加载过程  
1)、把一个PE文件读入内存缓冲区File->FileBuffer  
2)、PE文件的拉伸，FileBuffer->ImageBuffer  
3)、PE文件的还原，开辟另一块空间，把ImageBuffer重新还原成文件刚加载进内存时的样子  
4)、PE文件的转存，把3得到的结果转存为文件，看看是否可以运行

3、大体思路如下：  
1)、把一个PE文件读入内存缓冲区File->FileBuffer  
首先读取文件大小，根据文件大小开辟缓冲区，把缓冲区初始化0，然后把PE文件代码写入缓冲区，具体函数如下：  
ReadPEFile

```
LPVOID ReadPEFile(LPSTR lpszFile)
{
    FILE *pFile =NULL;
    //DWORD fileSize=0;
    LPVOID pFileBuffer=NULL;
    //打开文件
    pFile=fopen(lpszFile,"rb");
    if (!pFile)
    {
        printf("无法打开EXE文件!");
        return NULL;
    }
    //读取文件大小
    fseek(pFile,0,SEEK_END);
    fileSize=ftell(pFile);
    fseek(pFile,0,SEEK_SET);
    //分配缓冲区
    pFileBuffer=malloc(fileSize);
    if (!pFileBuffer)
    {
        printf("分配空间失败！");
        fclose(pFile);
        return NULL;
    }
    //将文件数据读取到缓冲区
    size_t n=fread(pFileBuffer,fileSize,1,pFile);
    if(!n)
    {
        printf("读取数据失败！");
        free(pFileBuffer);
        fclose(pFile);
        return NULL;
    }
    //关闭文件
    fclose(pFile);
    return pFileBuffer;
}
```

返回指向缓冲区的指针

2)、PE文件的拉伸  
拉伸前我们首先需要对PE文件的节表有一点的了解，PE文件的节表为一个结构体，具体如下：

```
typedef struct _IMAGE_SECTION_HEADER {
    BYTE    Name[IMAGE_SIZEOF_SHORT_NAME];
    union {
            DWORD   PhysicalAddress;
            DWORD   VirtualSize;
    } Misc;
    DWORD   VirtualAddress;    //内存中偏移地址
    DWORD   SizeOfRawData;    //PE文件中对其之后的大小
    DWORD   PointerToRawData;//为PE块区在PE文件中偏移
    DWORD   PointerToRelocations;
    DWORD   PointerToLinenumbers;
    WORD    NumberOfRelocations;
    WORD    NumberOfLinenumbers;
    DWORD   Characteristics;    //块区的属性：可读、可写..
} IMAGE_SECTION_HEADER, *PIMAGE_SECTION_HEADER;
```

首先我们要开辟一个新空间，大小为可选PE头中的SizeOfImage这个参数的值，然后一个节一个节的copy过去，FileBuffer中的PointerToRawData对应ImageBuffer中的VirtualAddress，copy的大小为SizeOfRawData，具体函数如下：  
CopyFileBufferToImageBuffer

```
LPVOID CopyFileBufferToImageBuffer()
{
    LPVOID pFileBuffer=NULL;
    LPVOID pImageBuffer=NULL;
    PIMAGE_DOS_HEADER pDosHeader=NULL;
    PIMAGE_NT_HEADERS pNTHeader=NULL;
    PIMAGE_FILE_HEADER pPEHeader=NULL;
    PIMAGE_OPTIONAL_HEADER32 pOptionHeader=NULL;
    PIMAGE_SECTION_HEADER pSectionHeader=NULL;
    pFileBuffer =ReadPEFile(FILEPATH);
    if(!pFileBuffer)
    {
        printf("文件读取失败\n");
        return 0;
    }
    //判断是否是有效的MZ标志
    if(*((PWORD)pFileBuffer)!=IMAGE_DOS_SIGNATURE)
    {
        printf("不是有效的MZ标志\n");
        free(pFileBuffer);
        return 0;
    }
    pDosHeader = (PIMAGE_DOS_HEADER)pFileBuffer;
 
    if(*((PDWORD)((DWORD)pFileBuffer+pDosHeader->e_lfanew))!=IMAGE_NT_SIGNATURE)
    {
        printf("不是有效的PE标志\n");
        free(pFileBuffer);
        return 0;
    }
    pNTHeader=(PIMAGE_NT_HEADERS)((DWORD)pFileBuffer+pDosHeader->e_lfanew);
     
    pPEHeader=(PIMAGE_FILE_HEADER)(((DWORD)pNTHeader)+4);
     
    //可选PE头
    pOptionHeader=(PIMAGE_OPTIONAL_HEADER32)((DWORD)pPEHeader+IMAGE_SIZEOF_FILE_HEADER);
    pImageBuffer=malloc(pOptionHeader->SizeOfImage);
    //初始化新空间
    memset(pImageBuffer,0,pOptionHeader->SizeOfImage);
    //copy头文件到新空间
    memcpy(pImageBuffer,pFileBuffer,pOptionHeader->SizeOfHeaders);
    pSectionHeader=(PIMAGE_SECTION_HEADER)((DWORD)pOptionHeader+pPEHeader->SizeOfOptionalHeader);
 
    //copy节
    for (int i=0;i<pPEHeader->NumberOfSections;i++,pSectionHeader++)
    {
        memcpy((void*)((DWORD)pImageBuffer+pSectionHeader->VirtualAddress),
            (void*)((DWORD)pDosHeader+pSectionHeader->PointerToRawData),pSectionHeader->SizeOfRawData);
     
    }
    //返回pImageBuffer
    return pImageBuffer;
}
```

3)、PE文件的还原和保存  
这一个没什么好说的，第二步的逆过程，直接上代码  
CopyImageBufferToNewBuffer

```
LPVOID CopyImageBufferToNewBuffer()
{
    LPVOID pImageBuffer=NULL;
    LPVOID pNewBuffer=NULL;
    PIMAGE_DOS_HEADER pDosHeader=NULL;
    PIMAGE_NT_HEADERS pNTHeader=NULL;
    PIMAGE_FILE_HEADER pPEHeader=NULL;
    PIMAGE_OPTIONAL_HEADER32 pOptionHeader=NULL;
    PIMAGE_SECTION_HEADER pSectionHeader=NULL;
    FILE *pFile =NULL;
    char *FILEPATH1="c:\\NOTEPAD1.EXE";
 
    pImageBuffer =CopyFileBufferToImageBuffer();
    pNewBuffer=malloc(fileSize);
    //初始化NewBuffer
    memset(pNewBuffer,0,fileSize);
 
    pDosHeader=(PIMAGE_DOS_HEADER)pImageBuffer;
    pNTHeader=(PIMAGE_NT_HEADERS)((DWORD)pDosHeader+pDosHeader->e_lfanew );
    pPEHeader=(PIMAGE_FILE_HEADER)(((DWORD)pNTHeader)+4);
    pOptionHeader=(PIMAGE_OPTIONAL_HEADER32)((DWORD)pPEHeader+IMAGE_SIZEOF_FILE_HEADER);
    memcpy(pNewBuffer,pImageBuffer,pOptionHeader->SizeOfHeaders);
    pSectionHeader=(PIMAGE_SECTION_HEADER)((DWORD)pOptionHeader+pPEHeader->SizeOfOptionalHeader);
    //copy 节
    for(int i=0;i<pPEHeader->NumberOfSections;i++,pSectionHeader++)
    {
        memcpy((void*)((DWORD)pNewBuffer+pSectionHeader->PointerToRawData),
            (void*)((DWORD)pImageBuffer+pSectionHeader->VirtualAddress),pSectionHeader->SizeOfRawData);
    }
    pFile=fopen(FILEPATH1,"wb");
    fwrite(pNewBuffer,fileSize,1,pFile);
 
    fclose(pFile);
    return 0;
}
```