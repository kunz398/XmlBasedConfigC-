# Making XML config-based application

## Create a class called xml.config

    public  class  XmlConfig
    
    {
    
    private  string  configFilePath;
    
    public  XmlConfig(string  configFilePath)
    
    {
    
    this.configFilePath  =  configFilePath;
    
    }
    
    public  class  Configurations
    
    {
    
    [XmlElement("ssh")]
    
    public  SshConfig  Ssh { get; set; }
    
    }
    
    public  class  SshConfig
    
    {
    
    public  string  Username { get; set; }
    
    public  string  Password { get; set; }
    
    public  string  Host { get; set; }
    
    public  string  RemoteDirectory { get; set; }
    
    }
    
    public  bool  CheckIfConfigExists()
    
    {
    
    try
    
    {
    
    // Check if the file exists
    
    if (File.Exists(configFilePath))
    
    {
    
    return  true;
    
    }
    
    else
    
    {
    
    MessageBox.Show("Please configure", "Config path not Defined", MessageBoxButtons.OK, MessageBoxIcon.Error);
    
    return  false;
    
    }
    
    }
    
    catch (Exception  ex)
    
    {
    
    MessageBox.Show("Error: "  +  ex.Message, "Exception for output path check", MessageBoxButtons.OK, MessageBoxIcon.Error);
    
    return  false;
    
    }
    
    }
    
    public  string  GetDecryptedPassword()
    
    {
    
    XmlDocument  xmlDoc  =  new  XmlDocument();
    
    xmlDoc.Load(configFilePath);
    
    XmlNode  passwordNode  =  xmlDoc.SelectSingleNode("/configurations/ssh/password");
    
    string  password  =  passwordNode.InnerText;
    
    string  secretKey  =  "H6F1A5B3D8C9F2F7E5D0B4C6A8F3C2D1";
    
    string  iv  =  "3B992BE41114A424"; // 16 bytes (32 hexadecimal characters)
    
    string  encryptedPassword  =  password.Substring("encrypted:".Length);
    
    string  decryptedText  =  Decrypt(encryptedPassword, secretKey, iv);
    
    return  decryptedText;
    
    }
    
    public  void  CheckIfPassIsEncrypted()
    
    {
    
    XmlDocument  xmlDoc  =  new  XmlDocument();
    
    xmlDoc.Load(configFilePath);
    
    XmlNode  passwordNode  =  xmlDoc.SelectSingleNode("/configurations/ssh/password");
    
    string  password  =  passwordNode.InnerText;
    
    if (password.StartsWith("encrypted:"))
    
    {
    
    // Password is already encrypted, do nothing.
    
    }
    
    else
    
    {
    
    string  secretKey  =  "H6F1A5B3D8C9F2F7E5D0B4C6A8F3C2D1";
    
    string  iv  =  "3B992BE41114A424"; // 16 bytes (32 hexadecimal characters)
    
    string  encryptedText  =  Encrypt(password, secretKey, iv);
    
    ModifyConfig("password", "encrypted:"  +  encryptedText);
    
    }
    
    }
    
    public  void  ModifyConfig(string  configModify, string  newValue)
    
    {
    
    try
    
    {
    
    XmlDocument  xmlDoc  =  new  XmlDocument();
    
    xmlDoc.Load(configFilePath); // Load the XML file
    
    // Dynamically build the XPath query based on the configModify value
    
    string  xpath  =  $"/configurations/ssh/{configModify}";
    
    // Find the node using the XPath query
    
    XmlNode  filepathNode  =  xmlDoc.SelectSingleNode(xpath);
    
    if (filepathNode  !=  null)
    
    {
    
    filepathNode.InnerText  =  newValue; // Replace with the desired value
    
    xmlDoc.Save(configFilePath); // Save the changes back to the XML file
    
    }
    
    else
    
    {
    
    MessageBox.Show($"Element '{configModify}' not found in the configuration.", "Error", MessageBoxButtons.OK, MessageBoxIcon.Error);
    
    }
    
    }
    
    catch (Exception  ex)
    
    {
    
    MessageBox.Show("Error: "  +  ex.Message, "Exception", MessageBoxButtons.OK, MessageBoxIcon.Error);
    
    }
    
    }
    
    private  string  Encrypt(string  plainText, string  key, string  iv)
    
    {
    
    Aes  aesAlg  =  Aes.Create();
    
    aesAlg.Key  =  Encoding.UTF8.GetBytes(key);
    
    aesAlg.IV  =  Encoding.UTF8.GetBytes(iv);
    
    ICryptoTransform  encryptor  =  aesAlg.CreateEncryptor(aesAlg.Key, aesAlg.IV);
    
    using (MemoryStream  msEncrypt  =  new  MemoryStream())
    
    {
    
    using (CryptoStream  csEncrypt  =  new  CryptoStream(msEncrypt, encryptor, CryptoStreamMode.Write))
    
    {
    
    using (StreamWriter  swEncrypt  =  new  StreamWriter(csEncrypt))
    
    {
    
    swEncrypt.Write(plainText);
    
    }
    
    }
    
    return  Convert.ToBase64String(msEncrypt.ToArray());
    
    }
    
    }
    
    private  string  Decrypt(string  cipherText, string  key, string  iv)
    
    {
    
    Aes  aesAlg  =  Aes.Create();
    
    aesAlg.Key  =  Encoding.UTF8.GetBytes(key);
    
    aesAlg.IV  =  Encoding.UTF8.GetBytes(iv);
    
    ICryptoTransform  decryptor  =  aesAlg.CreateDecryptor(aesAlg.Key, aesAlg.IV);
    
    MemoryStream  msDecrypt  =  new  MemoryStream(Convert.FromBase64String(cipherText));
    
    CryptoStream  csDecrypt  =  new  CryptoStream(msDecrypt, decryptor, CryptoStreamMode.Read);
    
    StreamReader  srDecrypt  =  new  StreamReader(csDecrypt);
    
    return  srDecrypt.ReadToEnd();
    
    }
    
    }

You can add your xml like so

    <?xml version="1.0" encoding="UTF-8"?>
    
    <configurations>
    
    <ssh>
    
    <username>readuser</username>
    
    <password>encrypted:TglhEGJeMdxOt0AC5jFeGA==</password>
    
    <host>127.0.0.1</host>
    
    <remoteDirectory>/home/kunz/archive</remoteDirectory>
    
    </ssh>
    
    </configurations>

This also ensures that the password is encrypted if you want to change password just remove everything between the password tag and then write your password in plain text the next time the application starts it will encrypt the password.

Also keep a note of

    ("/configurations/ssh....

This is configured according to the above XML edit this as needed.

Also change your

    string  secretKey  =  "H6F1A5B3D8C9F2F7E5D0B4C6A8F3C2D1";
    
    string  iv  =  "3B992BE41114A424"; // 16 bytes (32 hexadecimal characters)

to make it unique to you.

## Next using this in your Form

  

    public  partial  class  Form1 : Form
    
    {
    
    public  class  Configurations
    
    {
    
    [XmlElement("ssh")]
    
    public  SshConfig  Ssh { get; set; }
    
    }
    
    public  class  SshConfig
    
    {
    
    public  string  Username { get; set; }
    
    public  string  Password { get; set; }
    
    public  string  Host { get; set; }
    
    public  string  RemoteDirectory { get; set; }
    
    }
    
    private  const  string  ConfigDirectory  =  @"app_configs\app_name_config";
    
    private  const  string  ConfigFile  =  "config.xml";
    
    private  string  configFilePath;
    
    private  XmlConfig  xmlConfig;
    
    //configFilePath = Path.Combine(Environment.GetFolderPath(Environment.SpecialFolder.MyDocuments), ConfigDirectory, ConfigFile);
    
    public  Form1()
    
    {
    
    InitializeComponent();
    
    configFilePath  =  Path.Combine(Environment.GetFolderPath(Environment.SpecialFolder.MyDocuments), ConfigDirectory, ConfigFile);
    
    xmlConfig  =  new  XmlConfig(configFilePath); // Initialize XmlConfig with the configFilePath
    
    }
    
    private  void  Form1_Load(object  sender, EventArgs  e)
    
    {
    
    xmlConfig.CheckIfConfigExists();
    
    xmlConfig.CheckIfPassIsEncrypted();
    
    }
    
    }//end of form

I always store my configs in my documents,
