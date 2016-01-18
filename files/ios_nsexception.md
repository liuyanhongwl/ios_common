## NSException

#### 抓住崩溃

static void uncaughtExceptionHandler(NSException *exception) {
    
    NSLog(@"CRASH: %@", exception);
    
    NSLog(@"Stack Trace: %@", [exception callStackSymbols]);

    // Internal error reporting
    
}
//调用崩溃
NSSetUncaughtExceptionHandler(&uncaughtExceptionHandler);
