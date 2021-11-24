---
title: 网络组件-SHMNetworking
date: 2018-10-16 14:27:06
tags:
---

基于[AFNetworking](https://github.com/AFNetworking/AFNetworking)封装的网络库，同时参考了[YTKNetwork](https://github.com/yuantiku/YTKNetwork)，一个请求一个对象。

目前只实现了基本的发送http请求，上传、下载文件，基本满足彩云的需求。

## 常量定义

```objc
typedef NS_ENUM(NSUInteger, SHMHTTPMethod) {
    SHMHTTPMethodGET = 333,
    SHMHTTPMethodPOST,
    SHMHTTPMethodPUT,
    SHMHTTPMethodDELETE,
    // and more (HEAD, CONNECT, OPTIONS, TRACE)
};

/**
 * encode parameters and set default header (Content-Type, etc)
 */
typedef NS_ENUM(NSUInteger, SHMHRequestSerializerType) {
    /// query string for GET / form-encoded for POST, etc...
    SHMRequestSerializerTypeHTTP = 666,
    /// encode parameters as JSON
    SHMRequestSerializerTypeJSON,
};

/**
 * response data type you want.
 */
typedef NS_ENUM(NSUInteger, SHMResponseSerializerType) {
    /// I do NOT care about the responseObject
    SHMResponseSerializerTypeNone = 888,
    /// NSData
    SHMResponseSerializerTypeHTTP,
    /// JSON
    SHMResponseSerializerTypeJOSN,
    /// NSXMLParser
    SHMResponseSerializerTypeXMLParser,
};
```

## Protocol

### SHMHTTPRequesting

```objc
@protocol SHMHTTPRequesting <NSObject>

// 主机名
- (NSString *)host;

// 除去主机名部分的请求地址
- (NSString *)path;

// GET, POST, DELETE ...
- (SHMHTTPMethod)method;

// 请求参数，会根据method和requestSerializerType自动进行处理（GET时会自动拼接到URL上）
- (NSDictionary *)parameters;

- (NSDictionary *)httpHeader;
- (NSData *)bodyData;

- (BOOL)ignoreLocalCache;

// 返回的数据是否是压缩过的数据，返回YES时，会自动进行一次数据解压
- (BOOL)responseDataZipped;

// encode parameters and set default header (Content-Type, etc)
- (SHMHRequestSerializerType)requestSerializerType;

// response data type you want. 根据服务端返回的数据，设置对应的类型
- (SHMResponseSerializerType)responseSerializerType;

@optional
- (NSTimeInterval)timeout;

@end
```

### SHMDownloadRequesting

```objc
@protocol SHMDownloadRequesting <NSObject>

- (NSString *)host;
- (NSString *)path;
- (NSDictionary *)parameters;
- (NSDictionary *)httpHeader;
- (NSTimeInterval)timeout;

// 指定文件路径，下载到的数据会存到这个文件里
- (NSString *)destFilePath;

@end
```

### SHMUploadRequesting

```objc
@protocol SHMUploadRequesting <NSObject>

- (NSString *)host;
- (NSString *)path;
- (NSDictionary *)parameters;
- (NSDictionary *)httpHeader;
- (SHMResponseSerializerType)responseSerializerType;

// 以下两种方法用于提供要上传的数据
@optional
- (NSData *)fileData;
- (void(^)(id <AFMultipartFormData>))fileDataConstructor;

@end
```

## Demo

### 一般请求

新建一个request类，继承自类 **SHMHTTPRequest**，同时遵循协议 **SHMHTTPRequesting**。请求里需要的参数，通过实现协议里的方法提供。

```objc
// WDRolodexCreateGroupRequest.h

@interface WDRolodexCreateGroupRequest : SHMHTTPRequest <SHMHTTPRequesting>

- (instancetype)initWithGroupName:(NSString *)groupName orgId:(int64_t)orgId;

@end
```

```objc
// WDRolodexCreateGroupRequest.m

@implementation WDRolodexCreateGroupRequest {
    NSString *_groupName;
    int64_t _orgId;
}

- (instancetype)initWithGroupName:(NSString *)groupName orgId:(int64_t)orgId {
    self = [super init];
    if (!self) {
        return nil;
    }
    _groupName = groupName;
    _orgId = orgId;
    return self;
}

#pragma mark -- SHMHTTPRequesting

- (NSString *)host {
    return [WDRolodexRequestTools rolodexRequestHost];
}

- (NSString *)path {
    return @"group/create";
}

- (SHMHTTPMethod)method {
    return SHMHTTPMethodPOST;
}

- (NSDictionary *)parameters {
    return nil;
}

- (NSDictionary *)httpHeader {
    NSMutableDictionary *headers = [[NSMutableDictionary alloc] init];
    [headers addEntriesFromDictionary:[WDRolodexRequestTools rolodexRequestCommonHeadersWithOrgId:_orgId]];
    headers[@"orgId"] = @(_orgId).stringValue;
    return headers.copy;
}

- (NSData *)bodyData {
    NSMutableDictionary *bodyDic = [[NSMutableDictionary alloc] init];
    bodyDic[@"name"] = _groupName;
    NSData *bodyData = [NSJSONSerialization dataWithJSONObject:bodyDic options:NSJSONWritingPrettyPrinted error:nil];
    return bodyData;
}

- (BOOL)ignoreLocalCache {
    return YES;
}

- (BOOL)responseDataZipped {
    return NO;
}

- (SHMHRequestSerializerType)requestSerializerType {
    return SHMRequestSerializerTypeHTTP;
}

- (SHMResponseSerializerType)responseSerializerType {
    return SHMResponseSerializerTypeHTTP;
}

@end
```

### 下载

新建一个request类，继承自类 **SHMDownloadRequest**，同时遵循协议 **SHMDownloadRequesting**。请求里需要的参数，通过实现协议里的方法提供。

```objc
// WDDownloadFileRequest.h

@interface WDDownloadFileRequest : SHMDownloadRequest <SHMDownloadRequesting>

- (instancetype)initWithDownloadURL:(NSString *)downloadURL localFilePath:(NSString *)localFilePath;

- (instancetype)initWithDownloadURL:(NSString *)downloadURL localFilePath:(NSString *)localFilePath timeout:(NSTimeInterval)timeout;

@end
```

```objc
// WDDownloadFileRequest.m

@implementation WDDownloadFileRequest {
    NSString *_downloadURL;
    NSString *_localFilePath;
    NSTimeInterval _timeout;
}

- (instancetype)initWithDownloadURL:(NSString *)downloadURL localFilePath:(NSString *)localFilePath {
    return [self initWithDownloadURL:downloadURL localFilePath:localFilePath timeout:0];
}

- (instancetype)initWithDownloadURL:(NSString *)downloadURL localFilePath:(NSString *)localFilePath timeout:(NSTimeInterval)timeout {
    self = [super init];
    if (!self) {
        return nil;
    }
    _downloadURL = downloadURL;
    _localFilePath = localFilePath;
    _timeout = timeout;
    return self;
}

#pragma mark -- SHMDownloadRequesting

- (NSString *)host {
    return nil;
}

- (NSString *)path {
    return _downloadURL;
}

- (NSDictionary *)parameters {
    return nil;
}

- (NSDictionary *)httpHeader {
    return nil;
}

- (NSTimeInterval)timeout {
    return _timeout;
}

- (NSString *)destFilePath {
    return _localFilePath;
}

@end
```

一般情况下，彩云里可以直接使用 __WDDownloadFileRequest__ 来进行文件下载

### 上传

新建一个request类，继承自类 **SHMUploadRequest**，同时遵循协议 **SHMUploadRequesting**。请求里需要的参数，通过实现协议里的方法提供。

```objc
// WDUploadAvatarRequest.h

@interface WDUploadAvatarRequest : SHMUploadRequest <SHMUploadRequesting>

- (instancetype)initWithImageData:(NSData *)imageData;

@end
```

```objc
// WDUploadAvatarRequest.m

@implementation WDUploadAvatarRequest {
    NSData *_imageData;
}

- (instancetype)initWithImageData:(NSData *)imageData {
    self = [super init];
    if (!self) {
        return nil;
    }
    _imageData = imageData;
    return self;
}

#pragma mark -- SHMUploadRequesting

- (NSString *)host {
    return [[WDServerURLMgr sharedInstance] imHttpServerUrl];
}

- (NSString *)path {
    return [NSString stringWithFormat:@"/sfs/upload/avatar%@", [WDAvatarRequestTools requestToken]];
}

- (NSDictionary *)parameters {
    return nil;
}

- (NSDictionary *)httpHeader {
    return nil;
}

- (NSData *)fileData {
    return _imageData;
}

@end
```