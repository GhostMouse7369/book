## Kotlin-native+Gradle+rpi3B环境搭建

### 准备

Intelli IDEA工具

编译好的.so文件与.h文件，本文使用wiringpi库，使用bcm2835库同理，只是需要编译（[编译链接](<https://github.com/GhostMouse7369/book/blob/master/rpi/cmake%E7%BC%96%E8%AF%91bcm2835%E5%BA%93.md> )）



### 实现

#### 构建kotlin-native库

新建模块lib-wiring作为依赖库，也可直接配置在项目模块

###### build.gradle.kts

```kotlin
 linuxArm32Hfp {
        compilations["main"].apply {
            cinterops {
                create("libwiring")
            }
        }
 }
```

上面是cinterops最简单的构建配置，构建时会从工程src/nativeInterop/cinterop下查找libwiring.def文件配置，也可以把def相关配置在build.gradle.kts

详细配置参考：<https://www.kotlincn.net/docs/reference/building-mpp-with-gradle.html> 

导入.h文件到模块

![1571192019740](https://github.com/GhostMouse7369/book/blob/master/rpi/assets/1571192019740.png?raw=true)

配置def文件，指定需要的.h文件与.h文件路径

###### def文件

```ini
headers = wiringPi.h wiringPiI2C.h wiringSerial.h wiringPiSPI.h softPwm.h softServo.h softTone.h
#headerFilter = *
package = lib.wiring
compilerOpts = -Isrc/nativeInterop/c/include
```

#### 项目模块

新建项目模块led，实现PWM调光程序

导入.so文件到模块

![1571198367624](https://github.com/GhostMouse7369/book/blob/master/rpi/assets/1571198367624.png?raw=true)

###### build.gradle.kts

```kotlin
linuxArm32Hfp {
	//编译成可执行程序（rpi为Arm32架构）
    binaries.executable {
    	//程序名字
        baseName = "led"
        //程序主入口
        entryPoint("top.laoshuzi.rpi.led.main")
        //链接库文件
        linkerOpts = mutableListOf("-L/usr/local/lib", "-Llibs/linux_arm32", lwiringPi")
    }
    compilations["main"].apply {
        defaultSourceSet {
            dependencies {
                implementation(Deps.kotlin.stdlib.it)
                //依赖kotlin-native库
                implementation(project(":lib-wiring"))
            }
        }
    }
}
```

###### WiringPiPwmLedService.kt

```kotlin
class WiringPiPwmLedService : PwmLedService {

    private val pin = RPI_GPIO_P1_11.toInt()

    init {
        Log.d("wiringPiSetupGpio")
        wiringPiSetupGpio()
    }

    override fun openLed() {
        Log.d("softPwmCreate($pin,0,100)")
        softPwmCreate(pin, 0, 100)
    }

    override fun closeLed() {
        Log.d("softPwmStop($pin)")
        softPwmStop(pin)
    }

    override fun setLedLight(light: Float) {
        Log.d("softPwmWrite($pin,$light)")
        softPwmWrite(pin, light.toInt())
    }

}
```

###### Main.kt

```kotlin
fun main(args: Array<String>) {

    println("input[w|b]:")
    when (readLine()!!) {
        "w" -> wiringPiLed()
        "b" -> bcm2835Led()
        else -> throw Exception("input[w|b]")
    }

}

// S:P1-11 V:5v G:Gnd
fun wiringPiLed() {
    val ledService = WiringPiPwmLedService()
    while (true) {
        println("input[0-100]:")
        val light = readLine()?.toInt()!!
        when {
            light <= 0 -> ledService.closeLed()
            light >= 100 -> ledService.openLed()
            else -> ledService.setLedLight(light.toFloat() % 100)
        }
    }
}

// S:P1-12 V:5v G:Gnd
fun bcm2835Led() {
    Bcm2835.init()
    val ledService = PiPwmLedService()
    while (true) {
        println("input[0-100]:")
        val light = readLine()?.toInt()!!
        when {
            light <= 0 -> ledService.closeLed()
            light >= 100 -> ledService.openLed()
            else -> ledService.setLedLight(light.toFloat() % 100)
        }
    }
}
```



### 编译

linux系统可直接在Idea工具Gradle构建栏目运行

windows推荐使用linux子系统，然后cd到项目模块目录，执行对应的构建命令

```shell
# 根据gradle配置不同，后面的参数也不同，Release+Executable+LinuxArm32Hf
$ gradle linkReleaseExecutableLinuxArm32Hf
```

编译成功，在对应目录生成树莓派可执行文件

![1571199734547](https://github.com/GhostMouse7369/book/blob/master/rpi/assets/1571199734547.png?raw=true)

最后，拷贝到树莓派推荐用root权限运行

##### 
