# automated-plant-watering-project
# main.java
import edu.princeton.cs.introcs.StdDraw;
import org.firmata4j.I2CDevice;
import org.firmata4j.IODevice;
import org.firmata4j.Pin;
import org.firmata4j.firmata.FirmataDevice;
import org.firmata4j.ssd1306.SSD1306;
import java.io.IOException;
import java.util.ArrayList;
import java.util.Timer;
public class Main {
    static final byte I2C0 = 0x3C; // OLED Display
    public static void main(String[] args) throws IOException, InterruptedException {

        String myUSB = "/dev/cu.usbserial-0001";
        IODevice theArduinoObject = new FirmataDevice(myUSB);
        theArduinoObject.start();
        theArduinoObject.ensureInitializationIsDone();

        //Set up OLED
        I2CDevice i2cObject = theArduinoObject.getI2CDevice((byte) 0x3C);
        SSD1306 theOledObject = new SSD1306(i2cObject, SSD1306.Size.SSD1306_128_64);
        theOledObject.init();

        // Set up sensor and motor
        Pin sensor = theArduinoObject.getPin(15);
        sensor.setMode(Pin.Mode.ANALOG);
        Pin motor = theArduinoObject.getPin(7);
        motor.setMode(Pin.Mode.OUTPUT);

        ArrayList<Long> readings = new ArrayList<Long>();

        var task = new SensorTask(sensor, motor, theOledObject, readings);
        new Timer().schedule(task, 0, 1000);


        StdDraw.setXscale(-3,100);
        StdDraw.setYscale(-30,1100);
        StdDraw.setPenRadius(0.005);
        StdDraw.setPenColor(StdDraw.BLUE);
        StdDraw.line(0,0,0,1000);
        StdDraw.line(0,0,100,0);
        StdDraw.text(50,-30,"[X]");
        StdDraw.text(-3,500,"[Y]");
        StdDraw.text(50,1100,"Soil Sensor/Time Graph");

//       theArduinoObject.stop();
    }
}

#sensorTask.java
import edu.princeton.cs.introcs.StdDraw;
import org.firmata4j.firmata.FirmataDevice;
import org.firmata4j.ssd1306.SSD1306;
import org.firmata4j.ssd1306.MonochromeCanvas;
import org.firmata4j.I2CDevice;

import java.util.ArrayList;
import java.util.Timer;
import java.util.TimerTask;
import java.io.IOException;
import org.firmata4j.IODevice;
import org.firmata4j.Pin;

public class SensorTask extends TimerTask{

    private final SSD1306 theOledObject;
    private final Pin sensor;
    private final Pin motor;
    private ArrayList<Long> values;
    private int counter = 0;

    public SensorTask(Pin sensorPin, Pin motorPin, SSD1306 DisplayObject, ArrayList<Long> readings) {

        theOledObject = DisplayObject;
        sensor = sensorPin;
        motor = motorPin;
        values = readings;
    }
    @Override
    public void run() {
        counter++;
        // Sensor Object
        long sensorValue = sensor.getValue();
        values.add(sensorValue);

        StdDraw.text(counter, (double) values.get(values.size() - 1), ".");

        theOledObject.display();
        try{
            if(sensorValue>=730){
                this.motor.setValue(1);
                theOledObject.getCanvas().drawString(0,0, String.valueOf(sensorValue));
                theOledObject.getCanvas().drawString(0,50, "water pumping");
                theOledObject.display();
                System.out.println("water is pumping. Sensor value is: " + sensorValue);
            }
            else if (sensorValue<700) {
                this.motor.setValue(0);
                theOledObject.getCanvas().drawString(0,0, String.valueOf(sensorValue));
                theOledObject.getCanvas().drawString(0,50, "soil is wet");
                theOledObject.display();
                System.out.println("water stopped pumping. Sensor value is: " + sensorValue);
            }

        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

