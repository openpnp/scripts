# Feeder Groups

If your design has hundreds of discretes all in quantities of one’s or two, then a Blinds feeder is the most idle solution.
A rejected blank PCB with alignment holes for dowel pins is an ingenious Blinds feeder hosting up to a dozen strip feeders. The PnP job is broken down to _runs_ each run associated with its own Blinds Feeder.
Without going into the boring details this option of moving and loading the feeders along with deleting all “placements” and “parts” other than the current _run's_ clears clutter and makes placement more efficient to work on.
These two scripts
- **“1 Move Feeders (to disk).java”**
- **“2 Replace Feeders (from disk).java”**

Are the first two candidates in the long task of making a workflow convenient for small quantity PCB’s with very large number of unique components.

### To Do list:

1. _2 Replace Feeders (from disk).java_ updates from machine.xml. This is a stop gap arrangement. Park the machine to home and execute this script as it powers off the system, adds a bottom vision and possibly many other side unknown side effects!!

3. A single menu click to save the feeders of previous run to run_n1.feeder.xml file, delete all placements, delete all packages, loads run_n2.csv, loads feeders from run_n2.feeder.xml 

4. UI panel enhancement to aid this work flow.

#### 1 Move Feeders (to disk).java”
```
import java.io.File;
import java.io.FileWriter;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.nio.file.Files;
import org.openpnp.model.Configuration;
import org.openpnp.util;
import org.openpnp.spi.Feeder;

import javax.swing.JFileChooser;
import javax.swing.filechooser.FileNameExtensionFilter;

//System.out.println(System.getProperty("java.version"));

var config = Configuration.get();
config.save();                                      // else gui changes are lost!

File configDir = config.getConfigurationDirectory();
File macXml = new File (configDir, "machine.xml");


Path macXmlPathName = Paths.get(macXml.getAbsolutePath(), new String[0]);

//String str =  Files.readString(Path.of(file.getAbsolutePath())); -- not working!!
str = new String (Files.readAllBytes(macXmlPathName));
String[] arrOfStr = str.split("<feeders>|</feeders>");


//---- Save Feeders to Disk
var fileChooser = new JFileChooser();
fileChooser.setDialogTitle("Specify a file to save feeders");
fileChooser.setCurrentDirectory(new File(System.getProperty("user.home"), "\\Documents"));
//var ff = new FileNameExtensionFilter("XML","xml");
//fileChooser.addChoosableFileFilter(ff);
fileChooser.setAcceptAllFileFilterUsed(true);
 
int userSelection = fileChooser.showSaveDialog(null);
if (userSelection == JFileChooser.APPROVE_OPTION) {
    File fileToSave = fileChooser.getSelectedFile();
    System.out.println("Save as file: " + fileToSave.getAbsolutePath());

    //Files.writeString(Paths.get(fileToSave.getAbsolutePath(), new String[0]), arrOfStr[1], StandardOpenOption.CREATE); not working!!
    FileWriter fw = new FileWriter (fileToSave.getAbsolutePath());
    fw.write(arrOfStr[1]);
    fw.close();
    

    // That we have saved, delete all feeders to remove clutter -- this has side effects. Hence using deleteAllFeederrs
    //FileWriter fw = new FileWriter (macXml.getAbsolutePath());
    //fw.write(arrOfStr[0] + "\n<feeders>\n"+ "\n</feeders>\n" + arrOfStr[2]); 
    //fw.close();
    //config.load();      this is problematic

    deleteAllFeeders();
}
private void deleteAllFeeders(){

    feeders = machine.getFeeders();
    int j = feeders.size();
    //System.out.println(feeders.size());

    for (int i=j-1; i>=0; i--)
    {
        machine.removeFeeder(feeders.get(i));
    }
}
```


#### 2 Replace Feeders (from disk).java
```
import java.io.File;
import java.io.FileWriter;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.nio.file.Files;
import org.openpnp.model.Configuration;
import org.openpnp.util;

import javax.swing.JFileChooser;
import javax.swing.filechooser.FileNameExtensionFilter;

//System.out.println(System.getProperty("java.version"));

var config = Configuration.get();
File configDir = config.getConfigurationDirectory();
File macXml = new File (configDir, "machine.xml");

Path macXmlPathName = Paths.get(macXml.getAbsolutePath(), new String[0]);

//String str =  Files.readString(Path.of(file.getAbsolutePath()));  -- not working
str = new String (Files.readAllBytes(macXmlPathName));
String[] arrOfStr = str.split("<feeders>|</feeders>|<feeders/>");

int arrLen = arrOfStr.length;
 System.out.println(arrLen);

if (arrLen < 2) {
    javax.swing.JOptionPane.showMessageDialog(null, "maxhine.xml missing Feeders Node! Restore a backup");
    return;
}

JFileChooser fileChooser = new JFileChooser();
fileChooser.setCurrentDirectory(new File(System.getProperty("user.home"), "\\Documents"));
int result = fileChooser.showOpenDialog(null);
if (result == JFileChooser.APPROVE_OPTION) {
    File fileToOpen = fileChooser.getSelectedFile();

    Path feedPathName = Paths.get(fileToOpen.getAbsolutePath(), new String[0]);
    feeders = new String (Files.readAllBytes(feedPathName));

    FileWriter fw = new FileWriter (macXml.getAbsolutePath());
    fw.write(arrOfStr[0] + "\n<feeders>\n"+ feeders + "\n</feeders>\n" + arrOfStr[arrLen-1] );
    fw.close();

    config.load();
}
```
