# HTTPS on Tomcat

Enable HTTPS on your website running on Apache Tomcat on Windows Server 2019. Enabling HTTPS involves obtaining an SSL/TLS certificate and configuring Tomcat to use it. Below are the detailed steps to guide you through the process.

---

### **1. Obtain an SSL/TLS Certificate**

You have two options:

- **Option A: Create a Self-Signed Certificate** (suitable for testing or internal use)
- **Option B: Obtain a Certificate from a Trusted Certificate Authority (CA)** (recommended for production environments)

#### **Option A: Create a Self-Signed Certificate**

**a. Open Command Prompt as Administrator**

- Click on the Start menu, type `cmd`, right-click on **Command Prompt**, and select **Run as administrator**.

**b. Navigate to the Java `bin` Directory**

- Use the following command, replacing `<jdk_version>` with your installed Java version:

  ```shell
  cd "C:\Program Files\Java\jdk<jdk_version>\bin"
  ```

**c. Generate the Keystore and Self-Signed Certificate**

- Run the `keytool` command:

  ```shell
  keytool -genkeypair -alias tomcat -keyalg RSA -keysize 2048 -keystore C:\path\to\keystore.jks -validity 365
  ```

  - Replace `C:\path\to\keystore.jks` with the desired path for your keystore file.
  - You will be prompted to enter a password and provide certificate details (name, organization, etc.).

#### **Option B: Obtain a Certificate from a Trusted CA**

**a. Generate a Keystore**

- Similar to Option A, generate a keystore without creating a self-signed certificate:

  ```shell
  keytool -genkeypair -alias tomcat -keyalg RSA -keysize 2048 -keystore C:\path\to\keystore.jks -validity 365 -dname "CN=yourdomain.com, OU=YourUnit, O=YourOrg, L=YourCity, S=YourState, C=YourCountry"
  ```

**b. Create a Certificate Signing Request (CSR)**

- Generate the CSR file:

  ```shell
  keytool -certreq -alias tomcat -file C:\path\to\csr.csr -keystore C:\path\to\keystore.jks
  ```

**c. Submit the CSR to a CA**

- Send the `csr.csr` file to a trusted CA (e.g., Let's Encrypt, DigiCert).
- Follow their instructions to obtain the signed certificate files (usually in `.crt` or `.cer` format).

**d. Import the CA's Certificates into the Keystore**

- Import the root and intermediate certificates provided by the CA:

  ```shell
  keytool -import -alias root -keystore C:\path\to\keystore.jks -trustcacerts -file C:\path\to\root.crt
  keytool -import -alias intermediate -keystore C:\path\to\keystore.jks -trustcacerts -file C:\path\to\intermediate.crt
  ```

**e. Import Your Signed Certificate**

- Import your domain's certificate:

  ```shell
  keytool -import -alias tomcat -keystore C:\path\to\keystore.jks -file C:\path\to\your_certificate.crt
  ```

---

### **2. Configure Tomcat to Use the Keystore**

**a. Locate the `server.xml` File**

- Navigate to the `conf` directory in your Tomcat installation folder, e.g., `C:\Program Files\Apache Software Foundation\Tomcat\<version>\conf\server.xml`.

**b. Edit the `server.xml` File**

- Open `server.xml` with a text editor (e.g., Notepad++ or Visual Studio Code).

**c. Configure the SSL Connector**

- Find the existing SSL connector configuration. It might be commented out and look like this:

  ```xml
  <!--
  <Connector port="8443" protocol="org.apache.coyote.http11.Http11NioProtocol"
             maxThreads="150" SSLEnabled="true">
      <SSLHostConfig>
          <Certificate certificateKeystoreFile="conf/keystore.jks"
                       certificateKeystorePassword="changeit" />
      </SSLHostConfig>
  </Connector>
  -->
  ```

- Uncomment and modify it as follows:

  ```xml
  <Connector port="443" protocol="org.apache.coyote.http11.Http11NioProtocol"
             maxThreads="150" SSLEnabled="true" scheme="https" secure="true">
      <SSLHostConfig>
          <Certificate certificateKeystoreFile="C:\path\to\keystore.jks"
                       certificateKeystorePassword="your_keystore_password" />
      </SSLHostConfig>
  </Connector>
  ```

  - **Important Parameters**:
    - `port="443"`: Standard HTTPS port.
    - `certificateKeystoreFile`: Full path to your `keystore.jks`.
    - `certificateKeystorePassword`: The password you set when creating the keystore.

---

### **3. Open Port 443 in Windows Firewall**

**a. Open Windows Defender Firewall**

- Go to **Control Panel** > **System and Security** > **Windows Defender Firewall**.

**b. Create a New Inbound Rule**

- Click on **Advanced Settings**.
- Right-click on **Inbound Rules** and select **New Rule**.

**c. Configure the Rule**

- **Rule Type**: Select **Port**.
- **Protocol and Ports**:
  - **TCP** and **Specific local ports**: Enter `443`.
- **Action**: Allow the connection.
- **Profile**: Choose as per your network setup (Domain, Private, Public).
- **Name**: Give it a descriptive name like "Tomcat HTTPS Port 443".

---

### **4. Restart Tomcat**

**a. Stop the Tomcat Service**

- Open **Services** (type `services.msc` in the Run dialog).
- Find **Apache Tomcat**, right-click, and select **Stop**.

**b. Start the Tomcat Service**

- Right-click on **Apache Tomcat** again and select **Start**.

---

### **5. Test the HTTPS Connection**

- Open a web browser and navigate to `https://yourdomain.com`.
- If using a self-signed certificate, you may receive a security warning. Proceed if you trust the certificate.
- Your website should now be accessible over HTTPS.

---

### **Optional: Redirect HTTP Traffic to HTTPS**

To ensure all traffic uses HTTPS, you can configure a redirect:

**a. Modify `web.xml` in Your Application**

- In your web application's `WEB-INF` directory, edit `web.xml`.

**b. Add Security Constraint**

```xml
<security-constraint>
    <web-resource-collection>
        <web-resource-name>Protected Area</web-resource-name>
        <url-pattern>/*</url-pattern>
    </web-resource-collection>
    <user-data-constraint>
        <transport-guarantee>CONFIDENTIAL</transport-guarantee>
    </user-data-constraint>
</security-constraint>
```

---

### **Additional Considerations**

- **Keystore Formats**: Tomcat supports both JKS and PKCS12 keystore types. If you're using a PKCS12 file (e.g., `.pfx`), specify the keystore type:

  ```xml
  <Certificate certificateKeystoreFile="C:\path\to\keystore.pfx"
               certificateKeystorePassword="your_password"
               type="PKCS12" />
  ```

- **Java Version Compatibility**: Ensure that your Java version supports the cryptographic algorithms used in your certificates.

- **Intermediate Certificates**: If the CA provides intermediate certificates, ensure they are properly imported or included.

- **Firewall and Network Settings**: If you're behind a network firewall or proxy, additional configuration might be necessary.

---

### **Troubleshooting**

- **Check Tomcat Logs**: If Tomcat fails to start or HTTPS is not working, check the `catalina.out` or `localhost.log` files in the `logs` directory.
- **Common Errors**:
  - **Incorrect Keystore Password**: Ensure the password in `server.xml` matches the keystore password.
  - **Port Conflicts**: Verify that no other service is using port 443.
  - **Certificate Chain Issues**: Ensure all necessary certificates are imported into the keystore.
