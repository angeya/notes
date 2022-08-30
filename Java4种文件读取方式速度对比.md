# Java 4种文件读取方式速度对比

#### 通过字节流、缓冲字节流、随机文件访问、文件内存映射 计算 Git 安装包的 crc32校验和

```java
/**
 * 通过字节流、缓冲字节流、随机文件访问、文件内存映射 计算文件 crc32校验和
 * author: sunny
 * date: 2022/7/12 15:53
 */
public class CalReadFileTime {

    private static String filePath = "D:\\下载\\chrome\\Git-2.37.0-64-bit.exe";
    public static void main(String[] args) {
        inputStream();
        bufferedInputStream();
        randomAccessFile();
        fileMemoryMap();
    }

    /**
     * 普通字节流，一个字节一个字节从硬盘读取数据
     */
    private static void inputStream() {
        CRC32 crc32 = new CRC32();
        Instant start = Instant.now();
        try (InputStream inputStream = new FileInputStream(filePath)){
            int abyteValue;
            while ((abyteValue = inputStream.read()) != -1) {
                crc32.update(abyteValue);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        Duration duration = Duration.between(start, Instant.now());
        System.out.println("inputStream:" + crc32.getValue() + " duration:" + duration.toMillis());

    }

    /**
     * 带缓冲的字节流，没有数据时会从硬盘读取一块数据到缓冲区中
     */
    private static void bufferedInputStream() {
        CRC32 crc32 = new CRC32();
        Instant start = Instant.now();
        try (BufferedInputStream bufferedInputStream = new BufferedInputStream(new FileInputStream(filePath))){
            int abyteValue;
            while ((abyteValue = bufferedInputStream.read()) != -1) {
                crc32.update(abyteValue);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        Duration duration = Duration.between(start, Instant.now());
        System.out.println("bufferedInputStream:" + crc32.getValue() + "duration:" + duration.toMillis());

    }

    /**
     * 随机文件访问，也是一个字节一个字节读。因为提供了访问指针，所以理论上随度比 inputStream 慢
     */
    private static void randomAccessFile() {
        CRC32 crc32 = new CRC32();
        Instant start = Instant.now();
        try (RandomAccessFile randomAccessFile = new RandomAccessFile(filePath, "r")){
            int abyteValue;
            while ((abyteValue = randomAccessFile.read()) != -1) {
                crc32.update(abyteValue);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        Duration duration = Duration.between(start, Instant.now());
        System.out.println("randomAccessFile:" + crc32.getValue() + "duration:" + duration.toMillis());
    }

    /**
     * 文件内存映射，通过虚拟内存将整个或者部分文件内容加载到内存中
     */
    private static void fileMemoryMap() {
        CRC32 crc32 = new CRC32();
        Instant start = Instant.now();
        try (FileChannel fileChannel = FileChannel.open(Paths.get(filePath))){
            ByteBuffer byteBuffer = fileChannel.map(FileChannel.MapMode.READ_ONLY, 0, fileChannel.size());
            while (byteBuffer.hasRemaining()) {
                crc32.update(byteBuffer.get());
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        Duration duration = Duration.between(start, Instant.now());
        System.out.println("fileMemoryMap:" + crc32.getValue() + "duration:" + duration.toMillis());
    }
}
```

#### 结果对比

```verilog
inputStream:3199121536 duration:43084
bufferedInputStream:3199121536 duration:148
randomAccessFile:3199121536 duration:38037
fileMemoryMap:3199121536 duration:147
```

发现带缓存的输入流和文件内存映射这两种方式是最快的。