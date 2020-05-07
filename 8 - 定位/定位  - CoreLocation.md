## 定位  - CoreLocation

CoreLocation框架（CoreLocation.framework）可用于定位设备当前的经纬度，通过该框架，应用程序可通过附近的蜂窝基站、WIFI信号或者GPS等信息计算用户位置。
iOS SDK提供了CLLocationManager、CLLocationManagerDelegate来处理设备的定位信息，包括获取设备的方向以及进行方向检测等。
    

1.启动的时候获取定位权限

    [[CoreLocationTool shareInstance] locationWithBlock:^(CLLocation *location, CLLocation *baiduLocation, NSError *error) {
        if (error) {
            [UserInfo shareInfo].locationError = error;
        }else {
            if (baiduLocation != nil) {
                // 使用百度地址
                [UserInfo shareInfo].baiduLocation = baiduLocation;
            } else {
                [UserInfo shareInfo].baiduLocation = location;
            }
            [UserInfo shareInfo].location = location;
        }
        [CoreLocationTool handleLocaitonWithLoaction];
    }];

2.打开定位权限功能:   未打开权限就提醒用户打开权限，已打开就开始定位，已拒绝就提示用户已关闭定位功能：
    
    // 设置回调 
     - (void)locationWithBlock:(locationBlock)block {
    self.block = nil;
    //创建locationManager
    self.block = block;
    self.locatinoStatus = Locationing;
    if (![CLLocationManager locationServicesEnabled]) {
        ZCLog(@"你尚未打开定位,无法定位当前位置");
        self.locatinoStatus = LocationFailure;
        if (self.block) {
            NSDictionary *userInfo = @{NSLocalizedDescriptionKey:@"定位失败",NSLocalizedFailureReasonErrorKey:@"定位功能已关闭",NSLocalizedRecoverySuggestionErrorKey:@"请检查定位是否打开"};
            
            NSError *error = [[NSError alloc] initWithDomain:NSCocoaErrorDomain code:4 userInfo:userInfo];
            self.block(nil, nil, error);
            self.block = nil;
        }
        return;
    }
    
    //授权状态
    if ([CLLocationManager authorizationStatus] == kCLAuthorizationStatusAuthorizedWhenInUse || [CLLocationManager authorizationStatus] == kCLAuthorizationStatusAuthorizedAlways) {
        self.auth = YES;
    }else {
        self.auth = NO;
    }
    
    self.locationManager.desiredAccuracy = kCLLocationAccuracyBestForNavigation; // kCLLocationAccuracyBest;
    self.locationManager.distanceFilter = 1;
    //判断用户是否打开权限
    if (IOS8) {
        //打开权限  前台打开定位
        //如果没有授权则提醒用户授权
        //if ([CLLocationManager authorizationStatus] == kCLAuthorizationStatusNotDetermined) {
        [self.locationManager requestWhenInUseAuthorization];
        if (self.isAuthed == YES) {
            [self.locationManager startUpdatingLocation];
        }
        //}
    }else {
        [self.locationManager startUpdatingLocation];
    }
    }

3.处理定位成功后拿到的定位信息
   
     // 处理定位   
    + (void)handleLocaitonWithLoaction {
    if ([UserInfo shareInfo].locationError) {
        //定位失败
        NSDictionary *areaDic = [UtilTool userDefaultsGetObjectOfKey:kUserArea];
        if (areaDic) {
            //本地存在地区
            [UserInfo shareInfo].tempCityName = [NSString stringWithFormat:@"本地+%@", areaDic[@"area_name"]];
        }else {
            //本地不存在
            [UserInfo shareInfo].tempCityName = @"";
        }
    }else {
        //定位成功
        CLGeocoder *geocoder = [[CLGeocoder alloc] init];
        [geocoder reverseGeocodeLocation:[UserInfo shareInfo].location completionHandler:^(NSArray<CLPlacemark *> * _Nullable placemarks, NSError * _Nullable error) {
            if (!error) {
                CLPlacemark *placemark = [placemarks firstObject];
                if ([placemark.ISOcountryCode isEqualToString:@"CN"] && ([placemark.locality containsString:@"市"] || [placemark.locality containsString:@"特别行政区"])) {
                   if ([placemark.locality containsString:@"市"]) {
                       NSRange range = [placemark.locality rangeOfString:@"市"];
                       [UserInfo shareInfo].tempCityName = [placemark.locality substringToIndex:range.location];
                   } else {
                       // 特别行政区
                       NSRange range = [placemark.locality rangeOfString:@"特别行政区"];
                       [UserInfo shareInfo].tempCityName = [placemark.locality substringToIndex:range.location];
                       if ([[UserInfo shareInfo].tempCityName hasPrefix:@"中国"]) {
                           [UserInfo shareInfo].tempCityName = [[UserInfo shareInfo].tempCityName substringFromIndex:2];
                       }
                   }
                    
                    dispatch_queue_t queue = dispatch_queue_create("location", DISPATCH_QUEUE_CONCURRENT);
                    dispatch_async(queue, ^{
                        [self getLocationDistrictWithPlacemark:placemark];
                    });
                    
                    //发送通知
                    [[NSNotificationCenter defaultCenter] postNotificationName:kQXCityChangeNotification object:nil userInfo:@{@"from":@"LaunchVC"}];
                }else {
                    //国外
                    [UserInfo shareInfo].tempCityName = @"";
                    // [UserInfo shareInfo].vailableCity = NO;
                }
                
            }else {
                [UserInfo shareInfo].tempCityName = @"";
                // [UserInfo shareInfo].vailableCity = NO;
            }
        }];
    }
    }

4 解析定位信息   
       
       // 根据placemark 获取区县
    + (void)getLocationDistrictWithPlacemark:(CLPlacemark *)placemark {
    /**
         placemark.name 成汉南路43号
         placemark.thoroughfare 成汉南路
         placemark.subThoroughfare 43号
         placemark.locality 成都市
         placemark.subLocality 武侯区
         placemark.administrativeArea 四川省
         placemark.ISOcountryCode CN
         placemark.country 中国
     */
    // placemark.name 成汉南路43号 placemark.thoroughfare 成汉南路 placemark.subThoroughfare 43号 placemark.locality 成都市 placemark.subLocality 武侯区 placemark.administrativeArea 四川省 placemark.ISOcountryCode CN placemark.country 中国
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        NSString *areaPath = [[NSBundle mainBundle] pathForResource:@"area" ofType:@"plist"];
        NSArray *areasArr = [[NSArray alloc] initWithContentsOfFile:areaPath];
        for (NSDictionary *dic in areasArr) {
            if ([placemark.administrativeArea containsString:dic[@"name"]]) {
                QXAreaProvice *provice = [[QXAreaProvice alloc] init];
                [provice setValuesForKeysWithDictionary:dic];
                // 所在省
                for (QXAreaCity *city in provice.sub) {
                    if ([placemark.locality containsString:city.name]) {
                        // 所在市
                        for (QXAreaDistrict *district in city.sub) {
                            if ([placemark.subLocality containsString:district.name]) {
                                // 所在区
                                [UserInfo shareInfo].locationDistrict = [NSString stringWithFormat:@"%@ %@ %@", placemark.administrativeArea, placemark.locality, placemark.subLocality];
                                [UserInfo shareInfo].locationDistrict_Id = district.d_id;
                                break;
                            }
                        }
                    }
                }
            } else if (placemark.administrativeArea == nil && [placemark.locality containsString:dic[@"name"]]) {
               // 如果是直辖市 或者特别行政区
               QXAreaProvice *provice = [[QXAreaProvice alloc] init];
               [provice setValuesForKeysWithDictionary:dic];
               for (QXAreaCity *city in provice.sub) {
                       // 所在市
                       for (QXAreaDistrict *district in city.sub) {
                           if ([placemark.subLocality containsString:district.name]) {
                               // 所在区
                               [UserInfo shareInfo].locationDistrict = [NSString stringWithFormat:@"%@ %@", placemark.locality, placemark.subLocality];
                               [UserInfo shareInfo].locationDistrict_Id = district.d_id;
                               break;
                           }
                   }
               }
            }
        }
    });
   }

5 将定位相关的信息存储在用户信息（UserInfo）里面，需要的时候就可以直接查找出来

    /// 用户当前定位城市
    @property (nonatomic, strong, nullable) City *currentCity;
    /// 定位启动页从本地或定位获取的城市名字 请求成功后清除
    @property (nonatomic, strong, nullable) NSString *tempCityName;
    /// 是否手动选择城市
    @property (nonatomic, assign) BOOL handleSelectCity;
    /// 定位失败
    @property (nonatomic, strong, nullable) NSError *locationError;
    /// 百度地图的location
    @property (nonatomic, strong, nullable) CLLocation *baiduLocation;
    /// 系统地图的location
    @property (nonatomic, strong, nullable) CLLocation *location;
    /// 城市是否可用 如定位到国外则不可用
    @property (nonatomic, assign) BOOL vailableCity;
    /// 定位的区县
    @property (nonatomic, strong, nullable) NSString *locationDistrict;
    /// 定位的区县Id
    @property (nonatomic, strong, nullable) NSString *locationDistrict_Id;

