# Feeder Groups
If you feel changing _Part_ in the **Feeders** tab boring

**Or**

If your design has hundreds of discretes all in quantities of one’s or two, then you would probaly using Strip Feeders.
A rejected blank PCB with alignment holes for dowel pins is an ingenious Strip Feeder Holder hosting tens of feeders. As feeders increase feeder clutter also increase and now selecting the feeder and the right part is cluttered.
Without going into the boring details this option of saving and loading the feeders along with deleting all “placements” and “parts” other than the current _run's_ clears clutter and makes placement more efficient to work on.
These two scripts
- **“1 Open Feeders.java”**
- **“2 Save Feeders.java”**

are for saving and opening your feeder groups. "Open Feeders.java" replaces current feeders. If you do not want this functionality, just comment deleteAllFeeders();

#### “1 Open Feeders.java”
```
import org.openpnp.model.Configuration;
import org.openpnp.util.XmlSerialize;
import org.simpleframework.xml.Serializer;

import java.nio.file.Files;
import java.util.regex.Matcher;
import java.util.regex.Pattern;


private void OpenFeedersFromFile(){

    File fileToOpen = openFeeders();
    if (fileToOpen == null) 
        return;

    stringified_feeders = new String (Files.readAllBytes(fileToOpen.toPath()));

    

    Serializer serializer  = XmlSerialize.createSerializer();

    StringWriter strWr = new StringWriter();
    serializer.write(machine, strWr);     
    var stringified_machine = strWr.toString();

    String[] arrOfStr = stringified_machine.split("<feeders>|</feeders>|<feeders/>");

    int arrLen = arrOfStr.length;

    if ((arrLen < 2) || (arrLen > 3)) {
        javax.swing.JOptionPane.showMessageDialog(null, "maxhine.xml missing Feeders Node! Restore a backup");
        return;
    }

    stringified_machine = arrOfStr[0] + stringified_feeders + arrOfStr[arrOfStr.length-1];  // splice new feeders into machine.xml in memory

    StringReader sr = new StringReader(stringified_machine);
    machine2 = serializer.read(machine.getClass(), sr);                                     // deep copy of machne

    feeders = machine2.getFeeders();   
    int j = feeders.size();

    deleteAllFeeders();                                                                     // removing clutter


    for (int i=0; i<j; i++)
    {
        StringWriter sw = new StringWriter();
        feeder = feeders.get(i);
        serializer.write(feeder, sw);  

        // since part-id in feeder.class is not visibile...

        Pattern pattern = Pattern.compile("part-id=\"(.*?)\"");
        Matcher matcher = pattern.matcher(sw.toString());
        boolean matchFound = matcher.find();
        if(matchFound) {
            partId = matcher.group(1);
            part = config.getPart(partId);
            feeder.setPart(part);
            machine.addFeeder(feeder);        
        } else {
            print ("no part for" + feeder);
        }
    }
}



private File openFeeders() {
    FileDialog fileDialog = new FileDialog(gui, "Open Feeders...");
    fileDialog.setFilenameFilter(new FilenameFilter() {
        //@Override
        public boolean accept(File dir, String name) {
            return new name.toLowerCase().endsWith(".fed.xml"); 
        }
    });
    fileDialog.setVisible(true);
    try {
        String filename = fileDialog.getFile();
        if (filename == null) {
            return null;
        }
        if (!filename.toLowerCase().endsWith(".fed.xml")) {
            filename = filename + ".fed.xml"; 
        }
        File file = new File(new File(fileDialog.getDirectory()), filename);
        return file;
    }
    catch (Exception e) {
        MessageBoxes.errorBox(frame, "Error opening Feeders", e.getMessage());
        return null;
    }
}


private void deleteAllFeeders(){
    feeders = machine.getFeeders();
    int j = feeders.size();

    for (int i=j-1; i>=0; i--)
        machine.removeFeeder(feeders.get(i));
}

OpenFeedersFromFile();


```

#### “2 Save Feeders.java”
```
import org.openpnp.util.XmlSerialize;
import org.simpleframework.xml.Serializer;

import java.awt.FileDialog;

private void SaveFeedersToFile(){

    Serializer serializer  = XmlSerialize.createSerializer();

    StringWriter strWr = new StringWriter();
    serializer.write(machine, strWr);     
    var stringified_machine = strWr.toString();

    String[] arrOfStr = stringified_machine.split("<feeders>|</feeders>|<feeders/>");

    int arrLen = arrOfStr.length;

    if ((arrLen < 2) || (arrLen > 3)) {
        javax.swing.JOptionPane.showMessageDialog(null, "maxhine.xml missing Feeders Node! Restore a backup");
        return;
    }

    File fileToSave = saveFeedersAs();        //scripting.execute(scripting.getScriptsDirectory()+"\\0_b.java");

    if (fileToSave != null) {
        FileWriter fw = new FileWriter (fileToSave.getAbsolutePath());
        fw.write("<feeders>"+arrOfStr[1]+"</feeders>");
        fw.close();
    }
}

private File saveFeedersAs() {
    FileDialog fileDialog = new FileDialog(gui, "Save Feeders As...", FileDialog.SAVE);
    fileDialog.setFilenameFilter(new FilenameFilter() {
        //@Override
        public boolean accept(File dir, String name) {
            return new name.toLowerCase().endsWith(".fed.xml"); 
        }
    });
    fileDialog.setVisible(true);
    try {
        String filename = fileDialog.getFile();
        if (filename == null) {
            return null;
        }
        if (!filename.toLowerCase().endsWith(".fed.xml")) {
            filename = filename + ".fed.xml"; 
        }
        File file = new File(new File(fileDialog.getDirectory()), filename);
        return file;
    }
    catch (Exception e) {
        MessageBoxes.errorBox(frame, "Error saving Feeders", e.getMessage());
        return null;
    }
}


SaveFeedersToFile();

```

