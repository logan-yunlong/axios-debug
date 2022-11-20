---
sidebar_position: 1
---

# 使用readStream读取文件
一般情况下，对于大文件（例如10GB）我们会采用`流`的方式来读取文件，这样文件会被一点点的通过管道流动到对应的位置，从而减缓系统缓存的文件数量。

:::danger
注意：即使是使用了流的方式来读取文件，也并不代表可以高枕无忧，错误的处理流很有可能会导致内存的浪费。
:::
本文将会描述内存处理的各种边界情况


```javascript title="使用流的方式读取文件"
import fs from "fs";

// 定义一个方法来读取文件流
const readStream = fs.createReadStream('temp.csv');
// data 事件会不断触发
readStream.on('data', (chunk) => {
	console.log(readStream.bytesRead); //此时文件流共计读取到的数据
})
```

```javascript
import fs from "fs";

/**
 * 使用数据流写大文件
 * @param outPath
 * @return {Promise<void>}
 */
const writeBigFile = async function (outPath) {
    const writeStream = fs.createWriteStream(outPath);
    for (let i = 0; i < 1e8; i++) {
        // 写入内容
        let content = `${i},1\n`;
        const overWatermark = writeStream.write(content);
        writeStream.on('data', (chunk) => {
            console.log(chunk.length);
        })
        // false if the stream wishes for the calling code to wait for the 'drain' event to be emitted
        // before continuing to write additional data; otherwise true.
        // 如果stream希望调用者等待drain事件被触发
        if (!overWatermark) {
            console.log('发现overWatermark为' + overWatermark)
            await new Promise(resolve => {
                writeStream.once('drain', resolve)
            })
        }
    }
    writeStream.end();
}

writeBigFile('./temp/out.csv').then(r => {
    console.log(r)
})

```
