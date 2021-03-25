# Client Side Azure Storage Account Encryption Layer from DataStrata.io: Typescript
Typescript example of client-side encryption on Azure Storage using DataStrata.io Encryption Layers

## Prerequisites

Configure an Encryption Layer at DataStrata.io. [Here are some tips](https://datastrata.io/encryption-layer-overview-and-getting-started/) to get started with configuration.

## Getting Started

### From respository

You can download this example from:

https://github.com/DataStrata-io/encryption-layer-aws-s3-ts.git

1. Clone the repository: `git clone https://github.com/DataStrata-io/encryption-layer-aws-s3-ts.git`

2. Change into the directory: `cd encryption-layer-aws-s3-js`

3. Install the dependency: `npm install`

4. Replace `YOUR-REST-CREDENTIAL-CLIENT-ID` and `YOUR-REST-CREDENTIAL-SECRET` with the values you configured at [DataStrata.io: Getting Started with Encryption Layers](https://datastrata.io/encryption-layer-overview-and-getting-started/).

5. Replace `YOUR-AZURE-STORAGE-CONTAINER-NAME` with the name of the Container in your Storage Account.

6. Run `ts-node index.ts`

### From scratch

1. Create a directory and setup a Node.js project package.json. Quick one:

`npm init -y`

2. Install the npm package @Datastrata/.

`npm i @datastrata/aws-s3-encryption-layer`

3. Install typescript packages:

`npm install typescript --save-dev`

4. Add Node types:

`npm install @types/node --save-dev`

5. Add ts-node for compiling:

`npm install ts-node --save-dev`

6. Setup a tsconfig.json file:

`npx tsc --init --rootDir src --outDir lib --resolveJsonModule --lib es6,dom  --module commonjs`

7. Create an index.ts file with the following code:

        import { AzureEncryptionLayer } from '@datastrata/azure-blob-encryption-layer/src';
        import * as fs from 'fs';
        import * as crypto from 'crypto';
        
        const main = async () => {
            try {
                const testFileName = 'test-file.txt';
                const outputFilePath = 'downloaded-test-file.txt';
        
                const fileStream = fs.createReadStream(testFileName);
                fileStream.on('error', (err) => { console.log('File Error', err); });
        
                const encryptionLayer = new AzureEncryptionLayer(
                    'YOUR-REST-CREDENTIAL-CLIENT-ID',
                    'YOUR-REST-CREDENTIAL-SECRET',
                    'us-east-1');
        
                encryptionLayer.setContainerClient('YOUR-AZURE-STORAGE-CONTAINER-NAME');
                encryptionLayer.setBlockBlobClient(testFileName);
        
                const resultNoVersioningV1 = await encryptionLayer.uploadFile(testFileName);
                const downloadNoVersioningV1: any = await encryptionLayer.downloadToFile(outputFilePath);
        
                // Confirm file correctly downloaded and decrypted
                console.log('Uploaded file Sha1:  ', await getSha1(testFileName));
                console.log('Downloaded file Sha1:', await getSha1(outputFilePath));
        
                // Cleanup
                await encryptionLayer.delete();
                fs.unlinkSync(outputFilePath);
            } catch (e) {
                console.log(e);
            }
        }
        
        const getSha1 = async (path: string) => {
            return new Promise ( async (resolve, reject) => {
                try {
                    const fd = fs.createReadStream(path);
                    var hash = crypto.createHash('sha1');
                    hash.setEncoding('hex');
        
                    fd.on('end', function() {
                        hash.end();
                        const h = hash.read();
                        return resolve(h);
                    });
        
                    // read all file and pipe it (write it) to the hash object
                    fd.pipe(hash);
                } catch (e) {
                    console.log (e);
                    return reject(e);
                }
            });
        }
        
        main();

8. Replace `YOUR-REST-CREDENTIAL-CLIENT-ID` and `YOUR-REST-CREDENTIAL-SECRET` with the values you configured at [DataStrata.io: Getting Started with Encryption Layers](https://datastrata.io/encryption-layer-overview-and-getting-started/).

9. Replace `YOUR-AZURE-STORAGE-CONTAINER-NAME` with the name of the Container in your Storage Account.

10. Run the file by typing: `npx ts-node index.ts` (note that in some IDEs, just `ts-node index.ts` sufficient). You should see the contents of the uploaded file. If you used the `test-file.txt` in the repository, you will see:

> Welcome to DataStrata.io Encryption Layers, client-side encryption for your data.

