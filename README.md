# EliteQuant_Cpp
C/C++ high-frequency quantitative investment trading platform



## Project Outline

EliteQuant_Cpp is a multi-threaded concurrent high-frequency trading platform based on C/C++11. It follows modern design patterns such as event-driven, server/client architecture, dependency injection and loosely coupled powerful and stable distributed systems. It can run independently and be used directly. At the same time, it also serves as a server for other EliteQuant projects.

## Participate in development

We welcome contributions of any kind, including finding issues, sending code blocks, or creating pull requests. This also helps traders in other languages by sharing code architecture.

## Project installation

No installation is required, just download the code and use it.

The easiest way is to download and compile compiled.zip from the project root directory. Then run the program called eqsever.exe. Before running this executable, several configuration settings need to be modified. By default, the program reads config.xml from the same directory. So open the config file
1. If you want to use Interactive Brokers, please open Interactive Brokers Trading Platform (TWS), enter the menu File / Global Configuration / API / Settings, check "Enable ActiveX and Socket Client", uncheck "Read-Only API"
2. In the configuration file, change the Account ID to your own; the Interactive Brokers Account ID can usually be found at the top right of the TWS window.
3. If you use CTP, please change your brokerage account information and ctp address accordingly.
4. Create folders for log_dir and data_dir respectively. The former records the operation log, while the latter saves the time-sharing data

**Interactive Brokers**
is the most popular broker among retail traders. Quantopian, Quantconnect and many other retail trading platforms support IB. If you don't have an IB account, but want to try it out, they offer a demo account edemo with password demouser. Just download TWS Trader Workstation and log in with this demo account. Note that the account ID will change each time you log into the trading platform with a demo account, so you will have to change your EliteQuant configuration file accordingly.

**CTP**
It is the de facto standard of China's futures market, including commodity futures and financial futures. They also offer a free demo account [SimNow](http://simnow.com.cn). After registration, you will get account, password, brokerid, as well as market data and trading broker address. Replace it with the corresponding position of the EliteQuant configuration file.

## Development environment

Below is the environment we are using

* Visual Studio 2017 Community Edition on Windows
* CodeLite 11.0.6 on Linux

Visual C++ is a popular IDE on Windows. CodeLite is a free Linux IDE that is very close to Visual Studio in terms of user experience. Other options are CLION, CMake, etc.

### Development environment under Linux

You can follow the steps below to install the necessary third-party libraries and use cmake to build this project on the latest 64-bit Ubuntu system.

```bash
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install aptitude git cmake
sudo aptitude install zlib1g-dev rapidjson-dev python3-dev libboost-all-dev libsodium-dev \
                       libyaml-cpp-dev libwebsocketpp-dev libnanomsg-dev libzmq3-dev

# Download SimNow CTP tradeapi Linux version
cd ~ # or a directory of your choice
wget http://simnow.sfit.com.cn/download/api/v6.3.5_20150803_tradeapi_linux64.tar
tar xvf v6.3.5_20150803_tradeapi_linux64.tar
cd v6.3.5_20150803_api_tradeapi_linux64/
sudo cp thostmduserapi.so /usr/lib/libthostmduserapi.so
sudo cp thosttraderapi.so /usr/lib/libthosttraderapi.so
cd ~ # or a directory of your choice
git clone https://github.com/EliteQuant/EliteQuant_Cpp.git
cd EliteQuant_Cpp/source
mkdir build
cd build
cmake..
make -j2

# run the program
cd eqserver
cp ../../eqserver/config.yaml .
mkdir log data
./eqserver # Change config.yaml before this
```

On Linux, you may encounter a **double free or corruption(!prev)** error when pressing Ctrl+C to terminate eqserver. One way to suppress this warning is to add the MALLOC_CHECK_=0 variable to your environment.
```bash
sudo vim ~/.bashrc # Edit system configuration file
export MALLOC_CHECK_=0 # Add this line to the end of the file
source ~/.bashrc # reload configuration
```

## Project structure

### Microservices
| Service | Protocol | Port | Bundle or Connection |
| ------------- |:-------------:| -----:| -----:|
| MarketData | PUB | 55555 | Y |
| Brokerage | PAIR | 55556 | Y |
| DataManager/BarAggregator | PUB | 55557 | Y |
| TickRecording | SUB | 55555 | N |
| DataBoard | SUB | 55555 | N |
| ApiServer | PAIR | 55556 | N |
| ApiServer | SUB | 55557 | N |
| ApiServer | PAIR | 55558 | Y |

### Message protocol

Messages are separated by the character "|". For example

* New market order: o|account|API|customer order number|MKT|AAPL STK SMART|100[|order_flag]
* New limit order: o|account|API|customer order number|LMT|AAPL STK SMART|100|170.00[|order_flag]
* Transaction order status: s|account|API|service order number|customer order number|broker order number|order status
* Transaction: f|Account|API|Service Order No.|Customer Order No.|Broker Order No.|Deal No.|Transaction Time|Stock Code|Transaction Price|Transaction Quantity
* Cancellation command: c|account|API|service order number|customer order number|broker order number
* Time-sharing data: AAPL STK SMAR|time|data type|price|size|depth
* Time-sharing full data: AAPL STK SMART|Time|3|Price|Size|1|Buy 1|Buy 1 Amount|Sell 1|Sell 1 Amount|Open Interest|Open|High|Low|Last Close|Limit Price |Limit price

The following are message types:

* k: time-sharing data
* p: last price
* z: Last transaction volume
* o: new order
* c: cancel order
* s: order status
* n: position query
* m: general information
* b: K line
* h: historical data
* e: test message

### Trade item code

In EliteQuant, a trading product is identified by its full code **Full Symbol**, which consists of characters separated by spaces. The general pattern is "local symbol instrument type exchange multiplier"

* Stock: SPY STK SMART
* Futures: ESZ7 FUT GLOBEX 50
* Forex: EUR.USD CASH IDEALPRO
* Forex futures: 6BU1 FUT GLOBELX
* Option: GOOGL_140920P00535000 OPT SMART 100
* Futures options: EWQ4_C1730 FOP GLOBEX 50

### Order status
```cpp
enum OrderStatus {
OS_NewBorn = 0, // NewBorn
OS_PendingSubmit = 1,
OS_PendingCancel=2,
OS_Submitted = 3, // submitted
OS_Acknowledged = 4, // acknowledged
OS_Canceled = 5, // Canceled
OS_Filled = 6, // Filled
OS_Inactive = 7,
OS_PartiallyFilled = 8 // PartiallyFilled
};

enum OrderFlag { // for CTP offset flag
OF_OpenPosition = 0,
OF_ClosePosition = 1,
OF_CloseToday = 2,
OF_CloseYesterday = 3
};
```

### Code structure

![Code Structure](/resource/code_structure_cn.png?raw=true "Code Structure")

## Development Plan

* QT graphical interface: currently it is executed on the command side; plan to add user interface support.