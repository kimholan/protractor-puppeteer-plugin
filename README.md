# Protractor-puppeteer plugin

(!) This plugin can be used only for local run.
Running autotests in Docker, remote aren't supported yet. (TODO)

## Dependencies:
```
    "@types/puppeteer": "2.0.0"
    "chrome-har": "0.11.4",
    "puppeteer-core": "2.0.0"
```

## Requirements:
- Chrome >=76
- npm >=5.7.1
- node >=8.9.1

## How to add this plugin to protractor:

``` 
    // protractor.conf.js
    
    plugins: [
        {
            package: 'protractor-puppeteer-plugin',
            (or path: require.resolve('protractor-puppeteer-plugin'))
            configFile?: './path/to/puppeteer.conf.json',
            configOptions?: {
                connectToBrowser?: boolean, (Default: false)
                connectOptions?: {
                    defaultViewport?: {
                        width?: number, (Default: 800px)
                        height?: number, (Default: 600px)
                        deviceScaleFactor?: number, (Default: 1)
                        isMobile?: boolean, (Default: false)
                        hasTouch?: boolean, (Default: false)
                        isLandscape?: boolean (Default: false)
                    },
                    ignoreHTTPSErrors?: boolean, (Default: false)
                    slowMo?: number (Default: 0ms)
                },
                timeout?: number, (Default: 30000ms)
                harDir?: './path/to/artifatcs/dir/', (Default: './artifacts/har/')
                capture?: {
                    setRequestInterception: boolean, (Default: false)
                    logsDir?: './path/to/artifatcs/dir/', (Default: './artifacts/network/')
                    overrides?: {
                        url?: string,
                        method?: string,
                        postData?: string,
                        headers?: Object
                    },
                    requestfinished?: boolean, (Default: false)
                    requestfailed?: boolean, (Default: false)
                    response?: boolean (Default: false)
                }
            }
        }
    ]
```
(!) Note: The `configFile` property takes precedence over the `configOptions` property.

#### What should 'configFile' contain?

The `configFile` must be .json extension and contains the following properties.

E.g.:
```
    {
        connectToBrowser?: boolean, (Default: false)
        connectOptions?: {
           defaultViewport?: {
               width?: number, (Default: 800px)
               height?: number, (Default: 600px)
               deviceScaleFactor?: number, (Default: 1)
               isMobile?: boolean, (Default: false)
               hasTouch?: boolean, (Default: false)
               isLandscape?: boolean (Default: false)
           },
           ignoreHTTPSErrors?: boolean, (Default: false)
           slowMo?: number (Default: 0ms)
        },
        timeout?: number, (Default: 30000ms)
        harDir?: './path/to/artifatcs/dir/', (Default: './artifacts/har/')
        capture?: {
           setRequestInterception: boolean, (Default: false)
           logsDir?: './path/to/artifatcs/dir/', (Default: './artifacts/network/')
           overrides?: {
               url?: string,
               method?: string,
               postData?: string,
               headers?: Object
           },
           requestfinished?: boolean, (Default: false)
           requestfailed?: boolean, (Default: false)
           response?: boolean (Default: false)
        }
    }
```

#### Documentation
* `connectOptions`: https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#puppeteerconnectoptions
* `timeout`: https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#pagesetdefaulttimeouttimeout
* `capture`:
    * `request`: https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#class-request
    * `response`: https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#class-response


## How to use:

1. If you would like to connect to browser by yourself
or you would like to use some of the functions which returns from 'class: Puppeteer',
you should use `puppeteer` property:

    ```
        // myTest.js
        
        browser.puppeteer.launch([options]);
        browser.puppeteer.connect(options);
        browser.puppeteer.createBrowserFetcher([options]);
        browser.puppeteer.defaultArgs([options]);
        
        const iDevices = browser.puppeteer.devices['iDevices'];
    
        etc.
    ``` 
    More information about this class you can find here:
    * class: **Puppeteer**: https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#class-puppeteer
    
2. If Puppeteer was connected by protractor-puppeteer-plugin, you should use `channel` property.
The `channel` property provides to use all features of Puppeteer after merging with Protractor.

    ```
        // myTest.js
        
        browser.puppeteer.devices
   
        ...
        
        browser.channel.target
        
        browser.channel.client
        
        browser.channel.page
    
        browser.channel.browser
    ```
    More information about this class you can find here:
    * class: **Puppeteer**: https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#class-puppeteer
    * class: **Target**: https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#class-target
    * class: **CDPSession**: https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#class-cdpsession
    * class: **Page**: https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#class-page
    * class: **Browser**: https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#class-browser

3. For saving har files (with all calls from network) use:
    ```
   await browser.har.start();
   // test actions
   await browser.har.stop();
   ```

#### Example:
```
    // protractor.conf.js
    plugins: [
        {
            package: 'protractor-puppeteer-plugin',
            configFile: './path/to/puppeteer.conf.json'
        }
    ]

    // puppeteer.conf.json
    {
        connectToBrowser: true,
        connectOptions: {
            defaultViewport: {
                width: 1366,
                height: 768,
            }
        },
        timeout: 60000,
        capture: {
            setRequestInterception: true,
            requestfailed: true
        }
    }   
    
    // myTest.js
    describe('Suite name', () => {
        it('Test name', async () => {
            await browser.get('https://angular.io/');
            await browser.$('.hero-background a[href="https://angular.io/cli"]').click();
            await browser.channel.page.waitForNavigation({waitUntil: 'networkidle0'});
            await browser.channel.page.waitForSelector('#start', {visible: true, hidden: false});
            await browser.channel.page.goto('https://cli.angular.io/', {waitUntil: ['networkidle0', 'domcontentloaded']});
            await browser.channel.page.waitForResponse('https://cli.angular.io/favicon.ico');
            const getStartedBrn = browser.$('[href="https://angular.io/cli"]');
            await browser.wait(ExpectedConditions.visibilityOf(getStartedBrn));
            await getStartedBrn.click();

            expect(browser.$('aio-doc-viewer').isDisplayed()).to.eventually.equal(true, 'The "Get started" page was not opened');
        });
    });
```

### Documentation:
Protractor:
* https://www.protractortest.org
* https://github.com/angular/protractor

Puppeteer:
* https://pptr.dev/
* https://github.com/GoogleChrome/puppeteer
* https://try-puppeteer.appspot.com/
* https://developers.google.com/web/tools/puppeteer
* Examples:
    * https://developers.google.com/web/tools/puppeteer/examples

Chrome DevTools Protocol:
* https://chromedevtools.github.io/devtools-protocol/

### Contacts:
andreidei4ik@gmail.com
