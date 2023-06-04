# cspReport
import com.opencsv.CSVReader;
import com.opencsv.CSVWriter;
import com.opencsv.exceptions.CsvValidationException;
import java.io.File;
import java.io.FileNotFoundException;
import java.io.FileReader;
import java.io.FileWriter;
import java.io.IOException;
import java.util.HashMap;
import java.util.Map;
import org.apache.commons.lang3.StringUtils;
import org.json.JSONObject;

public class Main {
    final static String OUTPUT_FILE_DIRECTORY = "/Users/vedgaurav/repos/mutipleproj/proj1/cspreport";
    final static String CSV_REPORT = "/cspReport.csv";
    final static String CSV_REPORT2 = "/cspReport2.csv";
    static Map<String, String>  map = new HashMap<>();
    static Map<String, String>  map2 = new HashMap<>();
    public static void _loadCSPReport(){
        File outPutFile = new File(OUTPUT_FILE_DIRECTORY+CSV_REPORT);
        try {
            FileReader reader = null;
            CSVReader csvReader = null;
            if(outPutFile.exists()){
                reader = new FileReader(outPutFile);

                    csvReader = new CSVReader(reader);
                    csvReader.skip(1);
                    String[] nextRecord;
                    //System.out.println("loading started=========================================");
                    String key="";
                    while((nextRecord = csvReader.readNext()) != null){

                        //System.out.println(nextRecord[0] + " - " + nextRecord[1]);
                        key=nextRecord[0]+nextRecord[1];
                        map.put(key,nextRecord[1]);
                    }
                    System.out.println("File loading completed *********************************");
            }else{
                System.out.println("File not present ");
            }


        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } catch (CsvValidationException e) {
            e.printStackTrace();
        }
    }
    public static void _loadCSPReport2(){
        File outPutFile = new File(OUTPUT_FILE_DIRECTORY+CSV_REPORT2);
        try {
            FileReader reader = null;
            CSVReader csvReader = null;
            if(outPutFile.exists()){
                reader = new FileReader(outPutFile);
                csvReader = new CSVReader(reader);
                csvReader.skip(1);
                String[] nextRecord;

                while((nextRecord = csvReader.readNext()) != null){
                    map2.put(nextRecord[0],"");
                }
                System.out.println("File2 loading completed *********************************");
            }else{
                System.out.println("File2 not present ");
            }


        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } catch (CsvValidationException e) {
            e.printStackTrace();
        }
    }
        public static void main(String[] args) {
         final String FILE_DIRECTORY = "/Users/vedgaurav/repos/mutipleproj/proj1/logfiles";

         final String _CSPREPORT = "csp-report";
         final String _DOCUMENTURI = "document-uri";
         final String _EFFECTIVEDIRECTIVE = "effective-directive";
         final String _BLOCKEDURI = "blocked-uri";
         CSVWriter csvWriter = null;
         CSVWriter csvWriter2 = null;
         boolean updatedReport = false;
         boolean updatedReport2 = false;
        try {
            _loadCSPReport();
            _loadCSPReport2();
            File directorPath = new File(FILE_DIRECTORY);
            File[] fileList = directorPath.listFiles();
            File outPutFileDir = new File(OUTPUT_FILE_DIRECTORY);
            FileWriter outPutFileWriter = new FileWriter(outPutFileDir+CSV_REPORT,true);
            FileWriter outPutFileWriter2 = new FileWriter(outPutFileDir+CSV_REPORT2,true);
             csvWriter = new CSVWriter(outPutFileWriter);
             csvWriter2 = new CSVWriter(outPutFileWriter2);
            String[] outPutHeader = {"Directive", "BlockedURI"};
            String[] outPutHeader2 = {"BlockedURI"};
            if(map.isEmpty())
                csvWriter.writeNext(outPutHeader);
            if(map2.isEmpty())
                csvWriter2.writeNext(outPutHeader2);
            for(File file: fileList){
                System.out.println(file.getName());
                System.out.println(file.getAbsolutePath());
                FileReader fileReader = new FileReader(file);
                CSVReader csvReader = new CSVReader(fileReader);
                csvReader.skip(1);
                String[] nextRecord;
                while((nextRecord = csvReader.readNext()) != null){
                    String sub = StringUtils.substring(nextRecord[2],nextRecord[2].indexOf("{"),nextRecord[2].lastIndexOf("}")+1);
                    JSONObject json = new JSONObject(sub).getJSONObject(_CSPREPORT);
                    String effectiveDirective = json.getString(_EFFECTIVEDIRECTIVE);
                    String blockedURI = json.getString(_BLOCKEDURI);
                    //blockedURI = blockedURI.substring(0,blockedURI.lastIndexOf("/"));
                    blockedURI = blockedURI.substring(0,StringUtils.ordinalIndexOf(blockedURI,"/",3));
                    String[] data = {effectiveDirective, blockedURI};
                    if(!map.containsKey(effectiveDirective+blockedURI)){
                        csvWriter.writeNext(data);
                        updatedReport = true;
                        System.out.println(effectiveDirective+ " - "+blockedURI);
                        map.put(effectiveDirective+blockedURI,blockedURI);
                    }
                    if(!map2.containsKey(blockedURI)){
                        map2.put(blockedURI,"");
                        String[] data2={blockedURI};
                        csvWriter2.writeNext(data2);
                        updatedReport2=true;
                    }
                }
            }
            if(updatedReport){
                System.out.println("File was updated !!!!!!!!!!!!!!!!!!!!!!");
            }else{
                System.out.println("File was NOT updated --------------------");
            }
            if(updatedReport2){
                System.out.println("File2 was updated !!!!!!!!!!!!!!!!!!!!!!");
            }else{
                System.out.println("File2 was NOT updated --------------------");
            }

        }
            catch(Exception e){
                e.printStackTrace();
            }
        finally {
            try {
                if(csvWriter!=null)
                csvWriter.close();
                csvWriter2.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
