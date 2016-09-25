# Flow.js [![Build Status](https://travis-ci.org/flowjs/flow.js.svg)](https://travis-ci.org/flowjs/flow.js) [![Test Coverage](https://codeclimate.com/github/flowjs/flow.js/badges/coverage.svg)](https://codeclimate.com/github/flowjs/flow.js/coverage)

[![Saucelabs Test Status](https://saucelabs.com/browser-matrix/flowjs.svg)](https://saucelabs.com/u/flowjs)

Flow.jsは、HTML5 File APIを使って同時に複数のファイルのアップロードを安定かつレジューム可能に実現したJavaScriptライブラリです。
Flow.js is a JavaScript library providing multiple simultaneous, stable and resumable uploads via the HTML5 File API. 

このライブラリは、HTTPで大きなファイルをアップロードする際に耐障害性を持つことを目的として設計されました。これはファイルを小さな断片に分割することにより実現しています。つまり、断片のアップロードが失敗した場合にはいつも成功するまでアップロードがリトライされます。これによってローカルやサーバーへのネットワークのいずれかのネットワークが不調でもアップロードが自動的に再開されます。さらに、ユーザに一時停止、再開、さらに状態を損なわずにアップロードを回復することさえ可能です。これは、ファイル全体のアップロードが中止されるわけではなく、現在アップロード中の断片のみが中止されるためこのようなことが可能となっています。
The library is designed to introduce fault-tolerance into the upload of large files through HTTP. This is done by splitting each file into small chunks. Then, whenever the upload of a chunk fails, uploading is retried until the procedure completes. This allows uploads to automatically resume uploading after a network connection is lost either locally or to the server. Additionally, it allows for users to pause, resume and even recover uploads without losing state because only the currently uploading chunks will be aborted, not the entire upload.

FLow.jsは、`HTML5 File API`以外の外部依存がありません。これはファイルを小さな断片に分ける機能に基づいています。現在のところ、この方法がサポートされるのは、Firefox 4以上, Chrome 11以上, Safari 6以上 and Internet Explorer 10以上を限られています。
Flow.js does not have any external dependencies other than the `HTML5 File API`. This is relied on for the ability to chunk files into smaller pieces. Currently, this means that support is limited to Firefox 4+, Chrome 11+, Safari 6+ and Internet Explorer 10+.

サンプルは`samples/`フォルダにあります。このプロジェクトを助けるために、あなた自身のサンプルをMarkdownとして追加してください。
Samples and examples are available in the `samples/` folder. Please push your own as Markdown to help document the project.

## デモは見られますか？
[Flow.js + angular.js ファイルアップロードでデモ](http://flowjs.github.io/ng-flow/) - ng-flow 拡張ページ https://github.com/flowjs/ng-flow

JQueryとnode.jsバックエンドデモは、https://github.com/flowjs/flow.js/tree/master/samples/Node.js にあります。
JQuery and node.js backend demo https://github.com/flowjs/flow.js/tree/master/samples/Node.js

## インストールはどうしたら良いですか？

https://github.com/flowjs/flow.js/releases より最新ビルドをダウンロードしてください。
`dist/`フォルダに開発版と圧縮された製品版があります。
Download a latest build from https://github.com/flowjs/flow.js/releases
it contains development and minified production files in `dist/` folder.

または、bowerを使ってインストール：
```console
bower install flow.js#~2
```                
または、git cloneを使ってインストール：
```console
git clone https://github.com/flowjs/flow.js
```
## どうやって使えば良いですか？

何をどこへpostするかの情報を与えて新規に`Flow`オブジェクトを作成します：
A new `Flow` object is created with information of what and where to post:
```javascript
var flow = new Flow({
  target:'/api/photo/redeem-upload-token', 
  query:{upload_token:'my_token'}
});
// Flow.js isn't supported, fall back on a different method
if(!flow.support) location.href = '/some-old-crappy-uploader';
```
ファイルを選択させるかドラッグ&ドロップさせるために、ドロップのターゲットとブラウズするためにクリックされるDOMアイテムを割りあててください。
To allow files to be either selected and drag-dropped, you'll assign drop target and a DOM item to be clicked for browsing:
```javascript
flow.assignBrowse(document.getElementById('browseButton'));
flow.assignDrop(document.getElementById('dropTarget'));
```
その後、イベントを受け取ることでflow.jsとの対話を行います：
After this, interaction with Flow.js is done by listening to events:
```javascript
flow.on('fileAdded', function(file, event){
    console.log(file, event);
});
flow.on('fileSuccess', function(file,message){
    console.log(file,message);
});
flow.on('fileError', function(file, message){
    console.log(file, message);
});
```
## サーバーはどのように設定すれば良いのでしょうか？

flow.jsに関する魔法のほとんどはうユーザーのブラウザの中で起こりますが、サーバーサイドでは断片を再編成する必要があります。これは全く単純なタスク可能です。ですので、ファイルアップロードを受け取ることが可能ないかなるwebフレームワークや言語でも実装することが可能です
Most of the magic for Flow.js happens in the user's browser, but files still need to be reassembled from chunks on the server side. This should be a fairly simple task and can be achieved in any web framework or language, which is able to receive file uploads.

アップロード断片の状態を取り扱うため、すべてのリクエストには追加パラメタが一緒に送られます：
To handle the state of upload chunks, a number of extra parameters are sent along with all requests:

* `flowChunkNumber`: 現在のアップロードにおける断片のインデックス。最初n最初の断片は`1`となる (ここでは0始まりではない)。The index of the chunk in the current upload. First chunk is `1` (no base-0 counting here).
* `flowTotalChunks`: 断片の全体数。The total number of chunks.  
* `flowChunkSize`: 一般的な断片サイズ。この値と`flowTotalSize`を使って、断片の全体数を計算することができる。ただし、1ファイルにおける最後の断片では`flowChunkSize`より実際にHTTPで受診したデータは小さいかもしれない。The general chunk size. Using this value and `flowTotalSize` you can calculate the total number of chunks. Please note that the size of the data received in the HTTP might be lower than `flowChunkSize` of this for the last chunk for a file.
* `flowTotalSize`: 全体のファイルサイズ。The total file size.
* `flowIdentifier`: リクエストに含まれるそのファイルにユニークなID。A unique identifier for the file contained in the request.
* `flowFilename`: オリジナルファイル名（Firefoxのバグにより、マルチパートpostの断片ではファイル名が渡ってこない）。The original file name (since a bug in Firefox results in the file name not being transmitted in chunk multipart posts).
* `flowRelativePath`: ディレクトリを選択した場合にファイルの相対パス（Chromeを除くすべてのブラウザではディフォルトはファイル名）。The file's relative path when selecting a directory (defaults to file name in all browsers except Chrome).

同じ断片が２回以上アップロードされることを許してください。これは標準的な動きではないが、不安定なネットワークでは起こりえます。これはまさにFLow.jsが目指して設計されたケースです。
You should allow for the same chunk to be uploaded more than once; this isn't standard behaviour, but on an unstable network environment it could happen, and this case is exactly what Flow.js is designed for.

各リクエストでは、HTTPステータスコードを確認してください（`permanentErrors`オプションdオプションで変更可能）。
For every request, you can confirm reception in HTTP status codes (can be change through the `permanentErrors` option):

* `200`, `201`, `202`: 断片が受信できかつ正しい。再送不要。The chunk was accepted and correct. No need to re-upload.
* `404`, `415`. `500`, `501`: 断片がアップロードされたファイルはサポートされない。全体のアップロードをキャンセルする。The file for which the chunk was uploaded is not supported, cancel the entire upload.
* _Anything else_: 何らかの問題が発生。アップロードを再実行せよ。Something went wrong, but try reuploading the file.

## GET（または`test()`リクエスト）の処理

`testChunks`オプションを有効にすると、ブラウザを再起動したり別のブラウザに切り替えてアップロードを再開できます（理論的には、同じファイルのアップロードを複数のタブや異なるブラウザにまたがった状態でさせも行うことが可能）。`POST`データリクエストはFlow.jsにデータを受信させるために必要とされる。しかし、同じパラメタを持った対応するGETリクエストを実装するよう拡張することも可能：
Enabling the `testChunks` option will allow uploads to be resumed after browser restarts and even across browsers (in theory you could even run the same file upload across multiple tabs or different browsers).  The `POST` data requests listed are required to use Flow.js to receive data, but you can extend support by implementing a corresponding `GET` request with the same parameters:

* もしリクエストが`200`, `201` 及び `202` HTTPコードを返した場合は、断片は完了したものとみなされる。If this request returns a `200`, `201` or `202` HTTP code, the chunks is assumed to have been completed.
* もしリクエストが永久エラー状態を返したら、アップロードは停止される。If request returns a permanent error status, upload is stopped.
* もしリクエストがその他のコードを返したら、断片は通常通りアップロードされたものとする。If request returns anything else, the chunk will be uploaded in the standard fashion.

これが行われて`testChunks`が有効なら、ブラウザが再起動した後であっても、再アップロードの必要がないすでにアップロード済みの断片のチェックを行ってアップロードは直ちに追いつきます。
After this is done and `testChunks` enabled, an upload can quickly catch up even after a browser restart by simply verifying already uploaded chunks that do not need to be uploaded again.

## 全体のドキュメント　Full documentation

### Flow
#### Configuration

オブジェクトはコンフィグレーションオプションをつけてロードされる：
The object is loaded with a configuration options:
```javascript
var r = new Flow({opt1:'val', ...});
```
利用可能なコンフィグレーションオプションは：
Available configuration options are:

* `target` マルチパートPOSTリクエストに対するターゲットURL。これは文字列または関数です。関数の場合、引数としてFlowFile, FlowChunk, isTest(boolean)が渡されます。(ディフォルト: `/`)The target URL for the multipart POST request. This can be a string or a function. If a
function, it will be passed a FlowFile, a FlowChunk and isTest boolean (Default: `/`)
* `singleFile` 単一ファイルのアップロードを有効にする。いったん一つのファイルがアップロードされると、２番目のファイルは既存のファイルを追い越して、最初のファイルはキャンセルされます。(ディフォルト: false)Enable single file upload. Once one file is uploaded, second file will overtake existing one, first one will be canceled. (Default: false)
* `chunkSize` 各断片データのバイトサイズ。最後の断片は少なくてもこのサイズで２倍のサイズまでになりえます。その詳細と理由は、[Issue #51](https://github.com/23/resumable.js/issues/51)を参照してください。(ディフォルト: `1*1024*1024`)The size in bytes of each uploaded chunk of data. The last uploaded chunk will be at least this size and up to two the size, see [Issue #51](https://github.com/23/resumable.js/issues/51) for details and reasons. (Default: `1*1024*1024`)
* `forceChunkSize` すべての断片をchunkSize以下にします。そうでなければ、最後の断片は `chunkSize`以上になリます。(ディフォルト: `false`)Force all chunks to be less or equal than chunkSize. Otherwise, the last chunk will be greater than or equal to `chunkSize`. (Default: `false`)
* `simultaneousUploads` 同時アップロード数。(ディフォルト: `3`)Number of simultaneous uploads (Default: `3`)
* `fileParameterName` ファイル断片に使われるマルチパートPOSTパラメタの名前。(ディフォルト: `file`)　The name of the multipart POST parameter to use for the file chunk  (Default: `file`)
* `query` データ付きマルチパートPOSTに含める追加パラメタ。これはオブジェクトまたは関数です。関数の場合、引数としてFlowFile, FlowChunk, isTest(boolean)が渡されます。(ディフォルト: `{}`)　Extra parameters to include in the multipart POST with data. This can be an object or a
 function. If a function, it will be passed a FlowFile, a FlowChunk object and a isTest boolean
 (Default: `{}`)
* `headers` データ付きマルチパートPOSTに含める追加ヘッダー。関数の場合、引数としてFlowFile, FlowChunk, isTest(boolean)が渡されます。(ディフォルト: `{}`)Extra headers to include in the multipart POST with data. If a function, it will be passed a FlowFile, a FlowChunk object and a isTest boolean (Default: `{}`)
* `withCredentials` 標準CORSリクエストは、ディフォルトではいかなるクッキーの送信またはセットを行いません。リクエストの一部としてクッキーを含めたい場合は、`withCredentials`プロパティをtrueにセットしてください。(ディフォルト: `false`)Standard CORS requests do not send or set any cookies by default. In order to
 include cookies as part of the request, you need to set the `withCredentials` property to true.
(Default: `false`)
* `method` サーバーに断片をPOSTするときに使うメソッド（`multipart` or `octet`）。 (ディフォルト: `multipart`)Method to use when POSTing chunks to the server (`multipart` or `octet`) (Default: `multipart`)
* `testMethod` 断片がテストされるときに使われるHTTPメソッド。もし関数がセットされた場合は、引数としてFlowFile, FlowChunkが渡されます。(ディフォルト: `GET`)HTTP method to use when chunks are being tested. If set to a function, it will be passed a FlowFile and a FlowChunk arguments. (Default: `GET`)
* `uploadMethod` 断片がアップロードされるときに使われるHTTPメソッド。もし関数がセットされた場合は、引数としてFlowFile, FlowChunkが渡されます。(ディフォルト: `POST`)HTTP method to use when chunks are being uploaded. If set to a function, it will be passed a FlowFile and a FlowChunk arguments. (Default: `POST`)
* `allowDuplicateUploads ` いったんあるファイルがアップロードされると、同じファイルの再アップロードが許可されます。ディフォルトでは、もしあるファイルがすでにアップロードされている場合は、既存のFlowオブジェクトからそのファイルが取り除かれなければそのファイルはスキップされます。(ディフォルト: `false`)Once a file is uploaded, allow reupload of the same file. By default, if a file is already uploaded, it will be skipped unless the file is removed from the existing Flow object. (Default: `false`)
* `prioritizeFirstAndLastChunk` Prioritize first and last chunks of all files. This can be handy if you can determine if a file is valid for your service from only the first or last chunk. For example, photo or video meta data is usually located in the first part of a file, making it easy to test support from only the first chunk. (Default: `false`)
* `testChunks` Make a GET request to the server for each chunks to see if it already exists. If implemented on the server-side, this will allow for upload resumes even after a browser crash or even a computer restart. (Default: `true`)
* `preprocess` Optional function to process each chunk before testing & sending. To the function it will be passed the chunk as parameter, and should call the `preprocessFinished` method on the chunk when finished. (Default: `null`)
* `initFileFn` Optional function to initialize the fileObject. To the function it will be passed a FlowFile and a FlowChunk arguments.
* `readFileFn` Optional function wrapping reading operation from the original file. To the function it will be passed the FlowFile, the startByte and endByte, the fileType and the FlowChunk.
* `generateUniqueIdentifier` Override the function that generates unique identifiers for each file. (Default: `null`)
* `maxChunkRetries` The maximum number of retries for a chunk before the upload is failed. Valid values are any positive integer and `undefined` for no limit. (Default: `0`)
* `chunkRetryInterval` The number of milliseconds to wait before retrying a chunk on a non-permanent error.  Valid values are any positive integer and `undefined` for immediate retry. (Default: `undefined`)
* `progressCallbacksInterval` The time interval in milliseconds between progress reports. Set it
to 0 to handle each progress callback. (Default: `500`)
* `speedSmoothingFactor` Used for calculating average upload speed. Number from 1 to 0. Set to 1
and average upload speed wil be equal to current upload speed. For longer file uploads it is
better set this number to 0.02, because time remaining estimation will be more accurate. This
parameter must be adjusted together with `progressCallbacksInterval` parameter. (Default 0.1)
* `successStatuses` Response is success if response status is in this list (Default: `[200,201,
202]`)
* `permanentErrors` Response fails if response status is in this list (Default: `[404, 415, 500, 501]`)


#### Properties

* `.support` A boolean value indicator whether or not Flow.js is supported by the current browser.
* `.supportDirectory` A boolean value, which indicates if browser supports directory uploads.
* `.opts` A hash object of the configuration of the Flow.js instance.
* `.files` An array of `FlowFile` file objects added by the user (see full docs for this object type below).

#### Methods

* `.assignBrowse(domNodes, isDirectory, singleFile, attributes)` Assign a browse action to one or more DOM nodes.
  * `domNodes` array of dom nodes or a single node.
  * `isDirectory` Pass in `true` to allow directories to be selected (Chrome only, support can be checked with `supportDirectory` property).
  * `singleFile` To prevent multiple file uploads set this to true. Also look at config parameter `singleFile`.
  * `attributes` Pass object of keys and values to set custom attributes on input fields.
   For example, you can set `accept` attribute to `image/*`. This means that user will be able to select only images.
   Full list of attributes: http://www.w3.org/TR/html-markup/input.file.html#input.file-attributes

   Note: avoid using `a` and `button` tags as file upload buttons, use span instead.
* `.assignDrop(domNodes)` Assign one or more DOM nodes as a drop target.
* `.unAssignDrop(domNodes)` Unassign one or more DOM nodes as a drop target.
* `.on(event, callback)` Listen for event from Flow.js (see below)
* `.off([event, [callback]])`:
    * `.off()` All events are removed.
    * `.off(event)` Remove all callbacks of specific event.
    * `.off(event, callback)` Remove specific callback of event. `callback` should be a `Function`.
* `.upload()` Start or resume uploading.
* `.pause()` Pause uploading.
* `.resume()` Resume uploading.
* `.cancel()` Cancel upload of all `FlowFile` objects and remove them from the list.
* `.progress()` Returns a float between 0 and 1 indicating the current upload progress of all files.
* `.isUploading()` Returns a boolean indicating whether or not the instance is currently uploading anything.
* `.addFile(file)` Add a HTML5 File object to the list of files.
* `.removeFile(file)` Cancel upload of a specific `FlowFile` object on the list from the list.
* `.getFromUniqueIdentifier(uniqueIdentifier)` Look up a `FlowFile` object by its unique identifier.
* `.getSize()` Returns the total size of the upload in bytes.
* `.sizeUploaded()` Returns the total size uploaded of all files in bytes.
* `.timeRemaining()` Returns remaining time to upload all files in seconds. Accuracy is based on average speed. If speed is zero, time remaining will be equal to positive infinity `Number.POSITIVE_INFINITY`

#### Events

* `.fileSuccess(file, message, chunk)` A specific file was completed. First argument `file` is instance of `FlowFile`, second argument `message` contains server response. Response is always a string. 
Third argument `chunk` is instance of `FlowChunk`. You can get response status by accessing xhr 
object `chunk.xhr.status`.
* `.fileProgress(file, chunk)` Uploading progressed for a specific file.
* `.fileAdded(file, event)` This event is used for file validation. To reject this file return false.
This event is also called before file is added to upload queue,
this means that calling `flow.upload()` function will not start current file upload.
Optionally, you can use the browser `event` object from when the file was
added.
* `.filesAdded(array, event)` Same as fileAdded, but used for multiple file validation.
* `.filesSubmitted(array, event)` Same as filesAdded, but happens after the file is added to upload queue. Can be used to start upload of currently added files.
* `.fileRemoved(file)` The specific file was removed from the upload queue. Combined with filesSubmitted, can be used to notify UI to update its state to match the upload queue.
* `.fileRetry(file, chunk)` Something went wrong during upload of a specific file, uploading is being 
retried.
* `.fileError(file, message, chunk)` An error occurred during upload of a specific file.
* `.uploadStart()` Upload has been started on the Flow object.
* `.complete()` Uploading completed.
* `.progress()` Uploading progress.
* `.error(message, file, chunk)` An error, including fileError, occurred.
* `.catchAll(event, ...)` Listen to all the events listed above with the same callback function.

### FlowFile
FlowFile constructor can be accessed in `Flow.FlowFile`.
#### Properties

* `.flowObj` A back-reference to the parent `Flow` object.
* `.file` The correlating HTML5 `File` object.
* `.name` The name of the file.
* `.relativePath` The relative path to the file (defaults to file name if relative path doesn't exist)
* `.size` Size in bytes of the file.
* `.uniqueIdentifier` A unique identifier assigned to this file object. This value is included in uploads to the server for reference, but can also be used in CSS classes etc when building your upload UI.
* `.averageSpeed` Average upload speed, bytes per second.
* `.currentSpeed` Current upload speed, bytes per second.
* `.chunks` An array of `FlowChunk` items. You shouldn't need to dig into these.
* `.paused` Indicated if file is paused.
* `.error` Indicated if file has encountered an error.

#### Methods

* `.progress(relative)` Returns a float between 0 and 1 indicating the current upload progress of the file. If `relative` is `true`, the value is returned relative to all files in the Flow.js instance.
* `.pause()` Pause uploading the file.
* `.resume()` Resume uploading the file.
* `.cancel()` Abort uploading the file and delete it from the list of files to upload.
* `.retry()` Retry uploading the file.
* `.bootstrap()` Rebuild the state of a `FlowFile` object, including reassigning chunks and XMLHttpRequest instances.
* `.isUploading()` Returns a boolean indicating whether file chunks is uploading.
* `.isComplete()` Returns a boolean indicating whether the file has completed uploading and received a server response.
* `.sizeUploaded()` Returns size uploaded in bytes.
* `.timeRemaining()` Returns remaining time to finish upload file in seconds. Accuracy is based on average speed. If speed is zero, time remaining will be equal to positive infinity `Number.POSITIVE_INFINITY`
* `.getExtension()` Returns file extension in lowercase.
* `.getType()` Returns file type.

## Contribution

To ensure consistency throughout the source code, keep these rules in mind as you are working:

* All features or bug fixes must be tested by one or more specs.

* We follow the rules contained in [Google's JavaScript Style Guide](http://google-styleguide.googlecode.com/svn/trunk/javascriptguide.xml) with an exception we wrap all code at 100 characters.


## Installation Dependencies
1. To clone your Github repository, run:
```console
git clone git@github.com:<github username>/flow.js.git
```
2. To go to the Flow.js directory, run:
```console
cd flow.js
```
3. To add node.js dependencies
```console
npm install
```
## Testing

Our unit and integration tests are written with Jasmine and executed with Karma. To run all of the
tests on Chrome run:
```console
grunt karma:watch
```
Or choose other browser
```console
grunt karma:watch --browsers=Firefox,Chrome
```
Browsers should be comma separated and case sensitive.

To re-run tests just change any source or test file.

Automated tests is running after every commit at travis-ci.

### Running test on sauceLabs

1. Connect to sauce labs https://saucelabs.com/docs/connect
2. `grunt  test --sauce-local=true --sauce-username=**** --sauce-access-key=***`

other browsers can be used with `--browsers` flag, available browsers: sl_opera,sl_iphone,sl_safari,sl_ie10,sl_chrome,sl_firefox

## Origin
Flow.js was inspired by and evolved from https://github.com/23/resumable.js. Library has been supplemented with tests and features, such as drag and drop for folders, upload speed, time remaining estimation, separate files pause, resume and more.
