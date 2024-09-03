---
title: '"drain" event を使ってメモリリークを防ぐ'
category: "Coding"
tags: ["JavaScript", "Node.js"]
---

## 背景
あるcsvファイルのダウンロード処理で、下記のようなデータ加工して、csv streamに書き込む処理がある。

```js
// csv（配列データ）の作成
function　writeStream(data, writer) {
		data.forEach((dataRow) => {
			const result = processData(dataRow);
			writer.write(result);
		});
		writer.end();
	}

function processData(dataRow) {
    // データの加工処理
    return processedData;
  }
```

csvローデータが数万行以上の場合、データをDLする際にJavaScript heap out of memory　エラーが発生する。

```bash
2023-07-11T00:21:25: <--- Last few GCs --->
2023-07-11T00:21:25:
2023-07-11T00:21:25: [2482:0x605fd60] 18348258 ms: Scavenge 4033.7 (4126.2) -> 4033.4 (4134.2) MB, 24.7 / 0.0 ms  (average mu = 0.756, current mu = 0.490) allocation failure;
2023-07-11T00:21:25: [2482:0x605fd60] 18348307 ms: Scavenge 4040.3 (4134.2) -> 4040.2 (4134.2) MB, 38.3 / 0.0 ms  (average mu = 0.756, current mu = 0.490) allocation failure;
2023-07-11T00:21:25: [2482:0x605fd60] 18348350 ms: Scavenge 4041.1 (4134.2) -> 4040.3 (4150.2) MB, 41.9 / 0.0 ms  (average mu = 0.756, current mu = 0.490) allocation failure;
2023-07-11T00:21:25:
2023-07-11T00:21:25:
2023-07-11T00:21:25: <--- JS stacktrace --->
2023-07-11T00:21:25:
2023-07-11T00:21:25: FATAL ERROR: Reached heap limit Allocation failed - JavaScript heap out of memory
2023-07-11T00:21:25:  1: 0xb6b850 node::Abort() [/home/v18/.nvm/versions/node/v18.13.0/bin/node]
2023-07-11T00:21:25:  2: 0xa806a6  [/home/v18/.nvm/versions/node/v18.13.0/bin/node]
2023-07-11T00:21:25:  3: 0xd52140 v8::Utils::ReportOOMFailure(v8::internal::Isolate*, char const*, bool) [/home/v18/.nvm/versions/node/v18.13.0/bin/node]
2023-07-11T00:21:25:  4: 0xd524e7 v8::internal::V8::FatalProcessOutOfMemory(v8::internal::Isolate*, char const*, bool) [/home/v18/.nvm/versions/node/v18.13.0/bin/node]
2023-07-11T00:21:25:  5: 0xf2fbe5  [/home/v18/.nvm/versions/node/v18.13.0/bin/node]
2023-07-11T00:21:25:  6: 0xf420cd v8::internal::Heap::CollectGarbage(v8::internal::AllocationSpace, v8::internal::GarbageCollectionReason, v8::GCCallbackFlags) [/home/v18/.nvm/versions/node/v18.13.0/bin/node]
2023-07-11T00:21:25:  7: 0xf1c7ce v8::internal::HeapAllocator::AllocateRawWithLightRetrySlowPath(int, v8::internal::AllocationType, v8::internal::AllocationOrigin, v8::internal::AllocationAlignment) [/home/v18/.nvm/versions/node/v18.13.0/bin/node]
2023-07-11T00:21:25:  8: 0xf1db97 v8::internal::HeapAllocator::AllocateRawWithRetryOrFailSlowPath(int, v8::internal::AllocationType, v8::internal::AllocationOrigin, v8::internal::AllocationAlignment) [/home/v18/.nvm/versions/node/v18.13.0/bin/node]
2023-07-11T00:21:25:  9: 0xefed6a v8::internal::Factory::NewFillerObject(int, v8::internal::AllocationAlignment, v8::internal::AllocationType, v8::internal::AllocationOrigin) [/home/v18/.nvm/versions/node/v18.13.0/bin/node]
2023-07-11T00:21:25: 10: 0x12c265f v8::internal::Runtime_AllocateInYoungGeneration(int, unsigned long*, v8::internal::Isolate*) [/home/v18/.nvm/versions/node/v18.13.0/bin/node]
2023-07-11T00:21:25: 11: 0x16ef479  [/home/v18/.nvm/versions/node/v18.13.0/bin/node]
2023-07-11T00:22:05: [7] Worker died : [PID 2482] [Signal SIGABRT] [Code null]
```

だた、画面経由でなく、直接`curl`や`wget`でDLする場合は問題ない。

```bash
curl -o test.csv http://localhost:3000/api/download
```

最初は、axiosに問題があると思われたが、fetch APIでも同じエラーが発生した。

Heap Snapshotsを取得してみると、データ処理の途中データがメモリに大量に残っていることがわかった。

## 問題点
`data.forEach`でデータを処理しているが、データを一気にstreamに書き込みとしていることで、
フロントでのダウンロード処理が遅いと、メモリに保存するデータが膨らんでいき、メモリリークが発生する。

## 解決策

`drain`イベントを使って、streamが書き込み可能になったタイミングで次のデータを書き込むようにする。

```js
async function　writeStream(data, writer) {
  try {
	  for await (const dataRow of data) {
				const result = processData(dataRow);
					if (!writer.write(result)) {
						await new Promise((resolve) => writer.once("drain", resolve));
					}
			}
		} finally {
			writer.end();
		}
	}
```

`stream`自体がデーターの溜まり状況を判断できる。

`writable.write()`が`false`を返す場合は、`stream`に`buffer`溜まっていることを示しているため、
データーのforループを一時ストップさせ、書き込みを中止する。

`stream`の`buffer`がなくなったら(つまり`drain`になった)、次のループを継続し、書き込みを再開する。

ただ、なぜ`wget`を使うとき、メモリリークが発生しない点について、あくまで推測として、
`wget`はメモリに一時保存することなく、直接ファイルに書き込むため、ダウンロード処理スピードが`axios`や`fetch`より速いかもしれない。

また、`wget`が`HTTP/1.0` を使っている。`HTTP/1.0`は`HTTP/1.1`よりもシンプルで、影響あるかどうかは不明。

参照: 

[Using Heap Snapshot](https://nodejs.org/en/learn/diagnostics/memory/using-heap-snapshot)

[Node.js Stream　Event: 'drain'](https://nodejs.org/api/stream.html#event-drain)