public class TibcoSftpService {

    // These would ideally be in your application.properties
    @Value("${sftp.host}")
    private String host;
    
    @Value("${sftp.username}")
    private String username;

    @Value("${sftp.privateKeyPath}")
    private String privateKeyPath;

    @Value("${sftp.knownHostsPath}")
    private String knownHostsPath; // Path to your local known_hosts file

    public void downloadFilesSafely(String remoteDir, String localDir) {
        JSch jsch = new JSch();
        Session session = null;
        ChannelSftp channelSftp = null;

        try {
            // 1. Setup Private Key Authentication
            jsch.addIdentity(privateKeyPath);
            
            // 2. Load the Known Hosts file
            // This ensures the server's fingerprint matches what you expect
            jsch.setKnownHosts(knownHostsPath);

            session = jsch.getSession(username, host, 22);
            
            // Note: We REMOVED StrictHostKeyChecking=no. 
            // The default is now "yes", which requires a match in known_hosts.
            
            session.connect();
            channelSftp = (ChannelSftp) session.openChannel("sftp");
            channelSftp.connect();

            // 3. Process the "Disposable" File List
            Vector<ChannelSftp.LsEntry> fileList = channelSftp.ls(remoteDir);

            for (ChannelSftp.LsEntry entry : fileList) {
                String fileName = entry.getFilename();
                
                // Filter out current/parent directory references
                if (!entry.getAttrs().isDir() && !fileName.equals(".") && !fileName.equals("..")) {
                    
                    // Logic: Download immediately since 'ls' might have triggered deletion
                    System.out.println("Processing delta file: " + fileName);
                    
                    try {
                        channelSftp.get(remoteDir + "/" + fileName, localDir + "/" + fileName);
                        System.out.println("Successfully downloaded: " + fileName);
                    } catch (SftpException e) {
                        System.err.println("Failed to download " + fileName + ". It may have been purged already.");
                    }
                }
            }

        } catch (JSchException | SftpException e) {
            System.err.println("SSH/SFTP Error: " + e.getMessage());
        } finally {
            if (channelSftp != null) channelSftp.disconnect();
            if (session != null) session.disconnect();
        }
    }
}


----------------

- if no known_hosts needed
ession = jsch.getSession(username, host, port);
            session.setConfig("StrictHostKeyChecking", "no"); // Or provide known_hosts
            session.connect();


