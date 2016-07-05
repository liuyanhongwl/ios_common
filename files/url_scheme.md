## URL Scheme

Sometimes we need to open Setting's Preferences not of our app, but of the iPhone itself. What should we do to acomplish this?

![keyboard](https://cloud.githubusercontent.com/assets/724536/9033179/41e2d7be-39c5-11e5-8c25-8d123923ae94.gif)


 1. You must configure the URL Schemes in your project. You will find it in Target, Info, URL Scheme. Once there, just type **prefs** 

![gif-settings](https://cloud.githubusercontent.com/assets/724536/9033051/567a347a-39c4-11e5-9885-1e26460beab3.gif)

 2.- Later, just write the code with the URL path of the preference needed. In my case was the keyboard path.

##Swift 
    
     UIApplication.sharedApplication().openURL(NSURL(string:"prefs:root=General&path=Keyboard")!)

##Objective-c

    [[UIApplication sharedApplication] openURL:[NSURL URLWithString:@"prefs:root=General&path=Keyboard"]];


More info:

**And there you have all the url's available:**
    
| Command | Description |
| --- | --- |
| `prefs:` | The **topmost level** General |
| `prefs:root=General&path=About` | About |
| `prefs:root=General&path=ACCESSIBILITY` | Accessibility |
| `prefs:root=ACCOUNT_SETTINGS` | Account Settings |
| `prefs:root=AIRPLANE_MODE` | Airplane Mode |
| `prefs:root=General&path=AUTOLOCK` | Autolock |
| `prefs:root=Brightness` | Brightness |
| `prefs:root=General&path=Bluetooth` | Bluetooth iOS < 9 |
| `prefs:root=Bluetooth` | Bluetooth iOS > 9 |
| `prefs:root=CASTLE` | Castle |
| `prefs:root=General&path=USAGE/CELLULAR_USAGE` | Cellular Usage |
| `prefs:root=General&path=ManagedConfigurationList` | Configuration List|
| `prefs:root=General&path=DATE_AND_TIME` | Date and Time |
| `prefs:root=FACETIME` | Facetime |
| `prefs:root=General` | General |
| `prefs:root=INTERNET_TETHERING` | Internet Tethering|
| `prefs:root=MUSIC` | iTunes |
| `prefs:root=MUSIC&path=EQ` | iTunes Equalizer|
| `prefs:root=MUSIC&path=VolumeLimit` | iTunes Volume |
| `prefs:root=General&path=Keyboard` | Keyboard |
| `prefs:root=General&path=INTERNATIONAL` | Lang International |
| `prefs:root=LOCATION_SERVICES` | Location Services |
| `prefs:root=General&path=Network` | Network|
| `prefs:root=NIKE_PLUS_IPOD` | Nike iPod |
| `prefs:root=NOTES` | Notes|
| `prefs:root=NOTIFICATIONS_ID` | Notifications ID |
| `prefs:root=NOTIFICATIONS_ID&path=<app-scheme>`|Notifications App|
| `prefs:root=PASSBOOK` | Passbook |
| `prefs:root=Phone` | Phone|
| `prefs:root=Photos` | Photo Camera Roll |
| `prefs:root=General&path=Reset` | Reset |
| `prefs:root=Sounds&path=Ringtone` | Ringtone|
| `prefs:root=Safari` | Safari |
| `prefs:root=General&path=Assistant` | Siri|
| `prefs:root=Sounds` | Sounds |
| `prefs:root=General&path=SOFTWARE_UPDATE_LINK` | Software Update|
| `prefs:root=CASTLE&path=STORAGE_AND_BACKUP` | Storage & Backup |
| `prefs:root=STORE` | Store |
| `prefs:root=TWITTER` | Twitter|
| `prefs:root=General&path=USAGE` | Usage |
| `prefs:root=VIDEO` | Video|
| `prefs:root=General&path=Network/VPN` | VPN |
| `prefs:root=Wallpaper` | Wallpaper|
| `prefs:root=WIFI` | WIFI |

