1) Выделение выделение интерфейса, это позволит создавать тестовые классы или сделать тестовый бин
2) В данный момент класс жестко завязан на захардкоженный DEFAULT_FILE_NAME, что усложняет тестирование, можно сделать его параметром конструктора
3) Непонятно зачем нужна переменная sorted как Stream (или List)
4) Выделен отдельный FileProcessor, для непосредственной работы с файлами, он не должен содержать бизнесовую логику. Это упростит процесс тестирования за счет того что можно будет отдельно тестировать класс с бизнес логикой и класс обработчик файлов


public interface AuditManagerI {
    public void addRecord(String userName, LocalDateTime eventTime) throws IOException;
}

public class AuditManager implements AuditManagerI {

    private static final String DEFAULT_FILE_NAME = "audit_0.csv";
    private final int maxLinesPerFile;
    private final String directoryPath;
    private final FileProcessorI fileProcessor;

    private static int lastFilePathIndex = 0;

    public AuditManager(FileProcessorI fileProcessor,int maxLinesPerFile, String directoryPath) {
        this.maxLinesPerFile = maxLinesPerFile;
        this.directoryPath = directoryPath;
        this.fileProcessor = fileProcessor;
    }

    @Override
    public void addRecord(String userName, LocalDateTime eventTime) throws IOException {
        Path filePath = Path.of(directoryPath);

        String newRecord = userName + ";" + eventTime + "\n";

        if (filePath == null) {
            Path newFilePath = Path.of(directoryPath, DEFAULT_FILE_NAME);
            fileProcessor.process(newFilePath, newRecord, false);
            return;
        }

        List<String> lines = Files.readAllLines(filePath);
        if (lines.size() < maxLinesPerFile) {
            fileProcessor.process(newFilePath, newRecord, true);
        } else {
            lastFilePathIndex +=1;
            String newFileName = "audit_" + lastFilePathIndex + ".csv";
            Path newFilePath = Path.of(directoryPath, newFileName);
            fileProcessor.process(newFilePath, newRecord, false);
        }
    }
}

public interface FileProcessorI {
    public void process(Path targetFilePath, String newRecord, boolean needAppend) throws IOException;
}

public class FileProcessor implements FileProcessorI {
    @Override
    public void process(Path targetFilePath, String newRecord, boolean needAppend) throws IOException {
        if (needAppend) {
            Files.write(filePath, newRecord.getBytes(), StandardOpenOption.APPEND);
        } else {
            Files.write(targetFilePath, newRecord);
        }
    }
}
