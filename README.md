# blockchain-insurance-system-using-Node.js-and-Hyperledger-Fabric
    const fs = require('fs');
    const path = require('path');
    const ccpPath = path.resolve(__dirname, '..', '..', '..', 'first-network', 'connection.json');
    const ccp = JSON.parse(fs.readFileSync(ccpPath, 'utf8'));

    async function main() {
      const walletPath = path.join(process.cwd(), 'wallet');
      const wallet = await Wallets.newFileSystemWallet(walletPath);
      const identity = await wallet.get('user1');
      if (!identity) {
        console.log('An identity for the user "user1" does not exist in the wallet');
        console.log('Run the registerUser.js application before retrying');
        return;
      }

      const gateway = new Gateway();
      await gateway.connect(ccp, {
        wallet,
        identity: 'user1',
        discovery: { enabled: true, asLocalhost: true }
      });

      const network = await gateway.getNetwork('mychannel');
      const contract = network.getContract('insurance-contract');

      console.log('Submit Transaction: InitLedger');
      await contract.submitTransaction('InitLedger');
      console.log('Transaction has been submitted');

      console.log('Submit Transaction: CreatePolicy');
      await contract.submitTransaction('CreatePolicy', '1', 'laado bahadur', '1000', '2021-01-01', '2021-12-31', 'Auto', 'Car', '100000');
      console.log('Transaction has been submitted');

      console.log('Submit Transaction: MakeClaim');
      await contract.submitTransaction('MakeClaim', '1', '2021-05-01', '2021-05-30', '2500');
      console.log('Transaction has been submitted');

      console.log('Evaluate Transaction: GetPolicy');
      const policy = await contract.evaluateTransaction('GetPolicy', '1');
      console.log(`Policy: ${policy.toString()}`);

      console.log('Evaluate Transaction: GetClaims');
      const claims = await contract.evaluateTransaction('GetClaims', '1');
      console.log(`Claims: ${claims.toString()}`);

      await gateway.disconnect();
    }

    main();
    ```

    `registerUser.js`
    ```
    const fs = require('fs');
    const path = require('path');
    const ccpPath = path.resolve(__dirname, '..', '..', '..', 'first-network', 'connection.json');
    const ccp = JSON.parse(fs.readFileSync(ccpPath, 'utf8'));
    const fabricCaClient = require('fabric-ca-client');
    const { FileSystemWallet, Gateway } = require('fabric-network');

    async function main() {
      const walletPath = path.join(process.cwd(), 'wallet');
      const wallet = new FileSystemWallet(walletPath);
      const identity = await wallet.get('user1');
      if (identity) {
        console.log('An identity for the user "user1" already exists in the wallet');
        return;
      }

      const caInfo = ccp.certificateAuthorities['ca.org1.example.com'];
      const caTLSCACerts = caInfo.tlsCACerts.pem;
      const ca = new fabricCaClient(caInfo.url, {
        tlsClientCert: fs.readFileSync(path.join(walletPath, 'admin-cert.pem')),
        tlsClientKey: fs.readFileSync(path.join(walletPath, '
