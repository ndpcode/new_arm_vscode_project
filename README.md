# Create a new embedded ARM project in Visual Studio Code
Instruction for creating a new ARM embedded project in Visual Studio Code in Windows, for example creating project for STM32.

# Инструкция по созданию нового проекта embedded ARM в Visual Studio Code
Инструкция по настройке и развертыванию embedded C/C++ проекта для STM32 (и вообще любого ARM процессора), кросплатформенная (среда Linux или Windows) разработка в Visual Studio Code.
Т.к. разработка кросплатформенная, то в основе используются программы среды Linux, точнее программы инструментария GNU.
Для Linux настройка проще, поэтому ниже инструкция для Windows. *Инструкция от марта 2024 года.*

- **1. Установка инструментов GNU/Linux.**
    - 1.1. Необходим MSYS2, скачать с сайта msys2.org актуальную версию пакета и установить.
    - 1.2. В MSYS2 есть разные варианты системных приложений: clang, mingw, msys, ucrt. Используем mingw64. Сразу добавить путь к MinGW64 bin в "Дополнительные параметры системы\Переменные среды...\Системные переменные\Path", например это путь "C:\msys2\mingw64\bin\".
    - 1.3. Из каталога MSYS2 запустить "mingw64.exe", лучше с правами администратора. Откроется консольный терминал типа Bash.
    - 1.4. Для сборки проекта на основе Makefile необходимо приложение GNU make. Выполнить поиск пакета в терминале MinGW64 командой "pacman -Ss make", из результатов поиска выбрать make для 64 bit mingw, в моем случае это "mingw64/mingw-w64-x86_64-make". Установить командой "pacman -S mingw64/mingw-w64-x86_64-make".
    - 1.5. В каталоге "msys2\mingw64\bin\" появилось приложение "mingw32-make.exe". Для совместимости с командой "make" создать ссылку в Windows: в командной строке (cmd.exe) с правами администратора выполнить команду "mklink C:\msys2\mingw64\bin\make.exe C:\msys2\mingw64\bin\mingw32-make.exe". Ссылка также появится в каталоге "msys2\mingw64\bin\". Теперь при вводе команды "make" как в командной строке Windows, так и в терминале MinGW64 будет запускаться приложение "mingw32-make.exe".
    - 1.6. Для подключения к программатору и отладки программы на embedded плате необходимо приложение openocd. Установка openocd. Можно использовать как openocd, входящий в пакет MSYS2, так и openocd из проекта STMicroelectronics. Я в этой инструкции создаю проект для STM32, поэтому установлю openocd из проекта STMicroelectronics.
        - 1.6.1. Для установки openocd из MSYS2 выполнить в терминале MinGW64 поиск пакета командой "pacman -Ss openocd", из результатов поиска выбрать openocd для 64 bit mingw, в моем случае это "mingw64/mingw-w64-x86_64-openocd". Установить командой "pacman -S mingw64/mingw-w64-x86_64-openocd".
        - 1.6.2. Установка STMicroelectronics openocd.
            - 1.6.2.1. Репозиторий Git проекта STMicroelectronics openocd: https://github.com/STMicroelectronics/OpenOCD
            - 1.6.2.2. Для MinGW64, через "pacman -Ss" и "pacman -S" необходимо найти и установить пакеты: git, make, libtool, pkg-config, autotools, texinfo, libusb, hidapi, gcc. Если пакет недоступен для MinGW64, то установка аналогичного для MSYS2 - во время работы в MinGW64 автоматически подтянется версия от MSYS2.
            - 1.6.2.3. Работа в терминале MinGW64. Клонировать репозиторий "git clone https://github.com/STMicroelectronics/OpenOCD" в удобный каталог, путь к каталогу не должен содержать кириллицу и пробелы. Компиляция: сначала команда "./bootstrap", далее команда "./configure --host=x86_64-w64-mingw32" - это настройка сборки 64bit с использованием build tools mingw32. Затем сама сборка командой "make". Далее командой "make install" установка собранного openocd с копированием файлов в каталоги "\msys2\mingw64".
    - 1.7. Примечание: отладка здесь будет работать в связке VSCode + GDB + openocd.

- **2. Установка компилятора ARM embedded, открытого по лицензии типа GNU.**
    - 2.1. Скачать с сайта developer.arm.com Arm GNU Toolchain, например это "arm-gnu-toolchain-13.2.rel1-mingw-w64-i686-arm-none-eabi.exe". Для Windows на текущий момент доступна только 32 bit версия.
    - 2.2. Установить компилятор Arm GNU Toolchain, путь к компилятору не должен содержать пробелов. По завершении установки поставить "галочку" для создания переменной в Path Windows.
    - 2.3. Проверить в "Дополнительные параметры системы\Переменные среды...\Системные переменные\Path" наличие пути к компилятору типа "C:\Arm_GNU_Toolchain_arm-none-eabi\13.2_rel1\bin". Я переместил путь из пользовательских переменных в системные.

- **3. Для работы используется Visual Studio Code.** Установить Visual Studio Code, далее уже в нем установить расширения "C/C++", "C/C++ Extension Pack", "Makefile Tools", "Cortex-Debug", "MemoryView", "Peripheral Viewer". Эти расширения также автоматически установят дополнительные пакеты. По желанию еще можно установить "Arm Assembly" для работы с синтаксисом ассемблера ARM.

- **4. Отдельно должны быть установлены драйверы и программное обеспечение для программатора.**

- **5. Подготовка проекта C\C++ с исходным кодом для последующей работы в Visual Studio Code.**
    - 5.1. Начальную конфигурацию проекта программного обеспечения удобно производить в какой-либо специфической для платформы среде. Для STM32 установить STM32CubeMX. Создать новый проект в STM32CubeMX с необходимым микроконтроллером, настроить, целевую среду разработки выбрать "Makefile". Имя проекта не должно содержать пробелов и кириллицы, в том числе и внутри файла Makefile. Сгенерировать проект на базе Makefile.
    - 5.2. Если разработка в Windows, Makefile может содержать неправильную для Windows команду "clean". Открыть сгенерированный Makefile в каталоге проекта и внести следующее исправление:

было:
```
#######################################
# clean up
#######################################
clean:
	-rm -fR $(BUILD_DIR)
```
стало:
```
#######################################
# clean up
#######################################
#clean:
#	-rm -fR $(BUILD_DIR)
clean:
	rmdir /s /q $(BUILD_DIR)
```
-    - 5.3. Добавить в Makefile команду для прошивки программы, здесь указывается тип программатора, тип микроконтроллера или процессора, путь к файлу программы:
```
#######################################
# Download the application to flash
#######################################

prog:
	openocd -f interface/stlink.cfg -f target/stm32h7x.cfg -c "program $(BUILD_DIR)/$(TARGET).elf verify exit reset"
```
-    - 5.4. В каталоге проекта создать папку ".vscode". В ней создать файл "tasks.json" со следующим содержанием (таким образом создаются три задачи: Build, Clean, Program):
```json
{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "Build",
            "type": "shell",
            "group": "build",
            "command": "make",
            "problemMatcher": []
        },
        {
            "label": "Clean",
            "type": "shell",
            "group": "build",
            "command": "make clean",
            "problemMatcher": []
        },
        {
            "label": "Program",
            "type": "shell",
            "command": "make prog",
            "problemMatcher": []
        }
    ]
}
```

- **6. Настройка режима отладки проекта.**
    - 6.1. Для отладки необходим файл с описанием регистров микроконтроллера/процессора с расширением .svd. Для своего проекта на STM32 скачиваю файл "STM32H743.svd" с официального сайта STMicroelectronics и помещаю в корень проекта.
    - 6.2. В каталоге проекта ".vscode" создать файл "launch.json" со следующим содержанием (таким образом создается два сценария отладки: "Debug" для обычной отладки и "Debug (SWO console)" для отладки с поддержкой консоли SWO):
```json
{
	"version": "0.2.0",
	"configurations": [
		{
			"name": "Debug",
			"type": "cortex-debug",
			"request": "launch",
			"servertype": "openocd",
			"cwd": "${workspaceRoot}",			
			"executable": "build/${workspaceFolderBasename}.elf",
			"svdFile": "STM32H743.svd",
			"configFiles": [
				"interface/stlink.cfg",
				"target/stm32h7x.cfg"
			],
			"preLaunchTask": "Build"
		},
        {
            "name": "Debug (SWO console)",
            "type": "cortex-debug",
            "request": "launch",
            "servertype": "openocd",
            "cwd": "${workspaceRoot}",
            "executable": "build/${workspaceFolderBasename}.elf",
            "svdFile": "STM32H743.svd",
            "configFiles": [
				"interface/stlink.cfg",
				"target/stm32h7x.cfg"
			],
			"preLaunchTask": "Build",
            "swoConfig": 
            {
                "enabled": true,
                "cpuFrequency": 480000000,
                "swoFrequency": 2000000,
                "source": "probe",
                "decoders": [
                    {
                        "type": "console",
                        "port": 0,
                        "label": "output",
                        "encoding":"ascii"
                    }
                ]
            }
        }
	]
}
```
-    - 6.3. Для другого проекта с другим микроконтроллером/процессором поменять файл "STM32H743.svd" и строки "svdFile": "STM32H743.svd", "interface/stlink.cfg", "target/stm32h7x.cfg" на соответственно другие.

- **7. Для работы анализатора кода C/C++ в каталоге ".vscode" создать файл "c_cpp_properties.json" со следующим содержанием, которое также можно отредактировать под свой проект:**
```json
{
    "configurations": [
        {
            "name": "vscode_stm32_c_cpp_properties",
            "compilerPath": "arm-none-eabi-gcc",
            "includePath": [
                "${workspaceRoot}/**"
            ],
            "defines": [
                "STM32H743xx",
		"USE_HAL_DRIVER"
            ],
            "cStandard": "c11",
            "cppStandard": "c++17",
            "intelliSenseMode": "gcc-arm"
        }
    ],
    "version": 4
}
```
Здесь указываются каталоги включаемых файлов ("includePath"), блок define ("defines"), стандарты языка программирования. Указываемые значения должны соответствовать ключам сборки проекта в Makefile, чтобы анализ исходного кода в редакторе совпадал с анализом во время сборки/компиляции. 
- **8. Запустить Visual Studio Code. Открыть в Visual Studio Code каталог с проектом Makefile, сгенерированным в п.5. Расширение "Makefile Tools" автоматически анализирует проект.** 
- **9. В меню Visual Studio Code "Terminal" - "Run Task..." доступно три задачи: "Build", "Clean", "Program" - согласно настроек в п. 5.4 для соответственно сборки проекта, очистки проекта и загрузки прошивки в микроконтроллер.**
- **10. Во вкладке Visual Studio Code "Run and Debug" доступно два сценария отладки: "Debug" и "Debug (SWO console)" - согласно настроек в п. 6.2.**
- **11. На этом этапе уже можно работать с исходным кодом проекта. В первую очередь необходимо добавить следующие изменения.**
    - 11.1. Для активации отладочного порта SWO микроконтроллера необходимо в коде микроконтроллера создать и запустить один раз при инициализации (в начале функции main) следующую функцию:
```c
void SWO_ITM_enable(void)
{
  /*
    This functions recommends system speed of 480000000Hz and will
    use SWO clock speed of 2000000Hz

    # GDB OpenOCD commands to connect to this:
    monitor tpiu config internal - uart off 480000000
    monitor itm port 0 on

    Code Gen Ref: https://gist.github.com/mofosyne/178ad947fdff0f357eb0e03a42bcef5c
  */

  /* Setup SWO and SWO funnel (Note: SWO_BASE and SWTF_BASE not defined in stm32h743xx.h) */
  // DBGMCU_CR : Enable D3DBGCKEN D1DBGCKEN TRACECLKEN Clock Domains
  DBGMCU->CR =  DBGMCU_CR_DBG_CKD3EN | DBGMCU_CR_DBG_CKD1EN | DBGMCU_CR_DBG_TRACECKEN; // DBGMCU_CR
  // SWO_LAR & SWTF_LAR : Unlock SWO and SWO Funnel
  *((uint32_t *)(0x5c003fb0)) = 0xC5ACCE55; // SWO_LAR
  *((uint32_t *)(0x5c004fb0)) = 0xC5ACCE55; // SWTF_LAR
  // SWO_CODR  : 480000000Hz -> 2000000Hz
  // Note: SWOPrescaler = ((sysclock_Hz / SWOSpeed_Hz) - 1) --> 0x0000c7 = 239 = (480000000 / 2000000) - 1)
  *((uint32_t *)(0x5c003010)) = ((480000000 /  2000000) - 1); // SWO_CODR
  // SWO_SPPR : (2:  SWO NRZ, 1:  SWO Manchester encoding)
  *((uint32_t *)(0x5c0030f0)) = 0x00000002; // SWO_SPPR
  // SWTF_CTRL : enable SWO
  *((uint32_t *)(0x5c004000)) |= 0x1; // SWTF_CTRL
}
```
Функция SWO_ITM_enable() инициализирует порт SWO микроконтроллера STM32H743xx, для другого микроконтроллера STM32 или процессора может быть своя версия этой функции с настройкой других регистров по другим адресам (см. документацию).
-    - 11.2. В коде микроконтроллера необходимо переопределить низкоуровневый вывод данных в направление порта SWO. Для этого в исходный код микроконтроллера (например в файл main.c) добавить следующее переопределение функции _write:
```c
int _write(int file, char *ptr, int len)
{
  int DataIdx;

	for (DataIdx = 0; DataIdx < len; DataIdx++)
	{
		ITM_SendChar(*ptr++);
	}
	return len;
}
```
После этого вывод сообщений, например функцией printf(...), будет происходить в порт SWO.
- **12. Настройка порта SWO микроконтроллера STM32 для вывода отладочных сообщений в случае, если по вышеприведенной методике порт не заработал.**
    - 12.1. Созданная в п. 6.2 конфигурация "Debug (SWO console)" может не работать, соответственно во время отладки будут отсутствовать сообщения в терминале Visual Studio Code, т.е. будет висеть пустой терминал. Ниже привожу инструкцию по настройке порта SWO для микроконтроллера STM32, для настройки понадобится STM32CubeIDE. У меня получилось запустить порт SWO, скопировав настройки из STM32CubeIDE.
    - 12.2. Создать проект в STM32CubeIDE для своей модели микроконтроллера STM32, активировать порт SWO и скомпилировать проект (см. документацию STMicroelectronics). Настроить отладку проекта через "Debug probe = ST-LINK (OpenOCD)" и запустить отладку, STM32CubeIDE автоматически сгенерирует файлы конфигурации. Далее необходимо будет скопировать некоторые файлы из проекта STM32CubeIDE в проект Visual Studio Code.
    - 12.3. Необходимо настроить GDB + OpenOCD + Cortex-Debug VSCode для поддержки вывода через порт SWO. Создать каталог "swo_config" (можно каталог с другим именем) в корне проекта Visual Studio Code. В каталоге "swo_config" создать файл конфигурации "debug_with_swo.cfg" (можно свое имя файла) со следующим содержанием:
```
#----------------------------------------------
source [find interface/stlink-dap.cfg]

set WORKAREASIZE 0x8000

transport select "dapdirect_swd"

set CHIPNAME STM32H743BITx
set BOARDNAME genericBoard

# Enable debug when in low power modes
set ENABLE_LOW_POWER 1

# Stop Watchdog counters when halt
set STOP_WATCHDOG 1

# STlink Debug clock frequency
set CLOCK_FREQ 8000

# Reset configuration
# use hardware reset, connect under reset
# connect_assert_srst needed if low power mode application running (WFI...)
reset_config srst_only srst_nogate connect_assert_srst
set CONNECT_UNDER_RESET 1
set CORE_RESET 0

# ACCESS PORT NUMBER
set AP_NUM 0
# GDB PORT
#set GDB_PORT 50000

set DUAL_BANK 1


# BCTM CPU variables

source [find swo_config/stm32h7x.cfg]

# SWV trace
set USE_SWO 1
set swv_cmd "-protocol uart -output :3344 -traceclk 480000000"
source [find swo_config/swv.tcl]
#----------------------------------------------
```
Этот файл сгенерирован автоматически STM32CubeIDE. Для другого микроконтроллера необходимо взять этот файл в корне проекта STM32CubeIDE. Обратить внимание, что в этом файле находятся определения микроконтроллера и частот - при необходимости поменять на свои; ссылки на файлы "stm32h7x.cfg" и "swv.tcl" - необходимо прописать к ним путь через "swo_config", как сделано выше.
-    - 12.4. Файл stm32h7x.cfg (для другого микроконтроллера соответственно другой файл с другим именем) взять из установочного каталога STM32CubeIDE, например из каталога "C:\ST\STM32CubeIDE_1.14.1\STM32CubeIDE\plugins\com.st.stm32cube.ide.mcu.debug.openocd_2.1.0.202306221132\resources\openocd\st_scripts\target". Этот файл копируется в каталог "swo_config" проекта Visual Studio Code. Мне также потребовалось в начале файла "stm32h7x.cfg" прописать пути к файлам "mem_helper.tcl" и "gdb_helper.tcl" в каталоге "swo_config" (см. п.12.5).
     - 12.5. Файлы "mem_helper.tcl" и "gdb_helper.tcl" взять из установочного каталога STM32CubeIDE, например из каталога "C:\ST\STM32CubeIDE_1.14.1\STM32CubeIDE\plugins\com.st.stm32cube.ide.mcu.debug.openocd_2.1.0.202306221132\resources\openocd\st_scripts" и поместить в каталог "swo_config" проекта Visual Studio Code.
     - 12.6. Файл "swv.tcl" взять из установочного каталога STM32CubeIDE, например из каталога "C:\ST\STM32CubeIDE_1.14.1\STM32CubeIDE\plugins\com.st.stm32cube.ide.mcu.debug.openocd_2.1.0.202306221132\resources\openocd\st_scripts\board" и поместить в каталог "swo_config" проекта Visual Studio Code.
     - 12.7. Добавить в "\.vscode\launch.json" следующую конфигурацию:
```json
{
            "name":"Debug with SWO",
            "showDevDebugOutput": "raw",
            "preLaunchTask": "Build",
            "cwd": "${workspaceRoot}",
            "executable": "build/${workspaceFolderBasename}.elf",
            "request": "launch",
            "type": "cortex-debug",
            "servertype": "openocd",
            "device": "STM32H743BI",
			"configFiles": [
                "${workspaceRoot}/swo_config/debug_with_swo.cfg"
			],
            "svdFile": "STM32H743.svd",
            "interface": "swd",
            "swoConfig":{
                "enabled":true,
                "source":"probe",
                "swoFrequency": 2000000,
                "cpuFrequency":480000000,
                "decoders": [
                    {
                        "port": 0,
                        "type": "console",
                        "label": "SWO output",
                        "encoding":"ascii"
                    }
                ]
            }                
        }
```
Соответственно для другого микроконтроллера прописать другие имена в этом файле. Сохранить файл "launch.json" и убедиться, что конфигурация отладки "Debug with SWO" появилась в Visual Studio Code.
-    - 12.8. Запустить отладку с конфигурацией "Debug with SWO".
     - 12.9. Для вывода отладочных сообщений использовать стандартную функцию
```c
printf("Hello world\n");
```
и убедиться в наличии сообщений в окне консоли Visual Studio Code.
