# Making a simple volume plugin

## 0. Projucer setting

## 1. Adding an Audio Parameter to the PluginProcessor \<PluginProcessor\>
> Structure design: 
> 
> What are we making? - A volume plugin.\
> What do we need to change? - *a number* we can multiply to the signal **(gain)**.\
> What is the range of gain? - 0 (mute) ~ 1 (as-is) ~ 2(any amplifying number on your choice).

1. Declare a float pointer for gain.

    **📄PluginProcessor.h**
    ``` c++
    private:
        juce::AudioParameterFloat* mGainParam;
    ```
2. Instantiate the new PluginParameter in the PluginProcessor **constructor**.

   **📄PluginProcessor.cpp**
   ``` c++
   Mu45effectAudioProcessor::Mu45effectAudioProcessor()
   {
    addParameter(mGainParam = new juce::AudioParameterFloat("gain", // parameterID,
                                                            "Gain", // parameterName,
                                                            0.0f,   // minValue,
                                                            2.0f,   // maxValue,
                                                            1.0f)); // defaultValue
    }
   ```
3. We do not edit the **destructor** this time! Skip.

  **📄PluginProcessor.cpp**
   ``` c++
    Mu45effectAudioProcessor::~Mu45effectAudioProcessor()
    {
    }
   ```


> ### 🛠️ Build and Check time!
> See if a new paramater "Gain" appears in the plugin. We have not made the controller yet, so this parameter does not do anything yet!


1. Now, let's add a variable that stores a gain value as an *algorithm parameter*.

    **📄PluginProcessor.h**
    ``` c++
    private:
        // User Parameter
        juce::AudioParameterFloat* mGainParam;
        // Algorithm Parameter
        float mGainLinear;
    ```
2. Let's initialize `mGainLinear` value as its default `1.0` in `prepareToPlay()` method of the AudioProcessor. `PrepareToPlay()` runs before any kind of audio processing starts. 

    **📄PluginProcessor.cpp**
    ``` c++
    void Mu45effectAudioProcessor::prepareToPlay(double sampleRate, int samplesPerBlock)
    {
        mGainLinear = 1.0;
    }
   ```

3. We are now going to calculate the algorithm parameter and give `mGainLinear` its value. Let's define our *method* in the header file first.

    **📄PluginProcessor.h**
    ``` c++
    private:
        // User Parameter
        juce::AudioParameterFloat* mGainParam;
        // Algorithm Parameter
        float mGainLinear;
        // New Private User Method
        void calcAlgorithmParams();
    ```

4. If we call this method, it will calculate the value of the algorithm parameter based on user parameter value. In this case, **user parameter value == algorithm parameter value**. Before the `processBlock()` method, let's add and fill our new method, `calcAlgorithmParams()`.

    **📄PluginProcessor.cpp**
    ``` c++
    void Mu45effectAudioProcessor::calcAlgorithmParams()
    {
        mGainLinear = mGainParam->get();
    }
   ```

5. Now, we process the audio signal with our gain value in `processBlock()`.   

    **📄PluginProcessor.cpp**
    ``` c++
    void Mu45effectAudioProcessor::processBlock(juce::AudioBuffer<float> &buffer, juce::MidiBuffer &midiMessages)
    {
        // Some JUCE book-keeping stuff
        juce::ScopedNoDenormals noDenormals;
        auto totalNumInputChannels = getTotalNumInputChannels();
        auto totalNumOutputChannels = getTotalNumOutputChannels();
        for (auto i = totalNumInputChannels; i < totalNumOutputChannels; ++i)
            buffer.clear(i, 0, buffer.getNumSamples());

        // Update the algorithm params for the user params before processing the audio
        calcAlgorithmParams();

        // Get the left and right audio buffers
        auto* channelDataLeft = buffer.getWritePointer(0);
        auto* channelDataRight = buffer.getWritePointer(1);

        // Step through each sample in the audio buffer
        for (int samp = 0; samp < buffer.getNumSamples(); samp++)
        {
            // process each audio sample
            channelDataLeft[samp] = mGainLinear * channelDataLeft[samp];
            channelDataRight[samp] = mGainLinear * channelDataRight[samp];
        }
    }
   ```
> ### 🛠️ Build and Check time!
> See if a the volume changes as you change the Gain parameter in your DAW's default UI.  We have not made a GUI for the plugin yet, but the parameter change in the DAW will change the volume!

 
## 2. Making a Slider GUI \<PluginEditor\>
1. Add a Slider object in the header file.

   **📄PluginEditor.h**
   ``` c++
    private:
        juce::Slider mVolumeSlider;
   ```

2. Set some parameters of the Slider object.
    > Think of the volume slider in your computer. What does it have?\
    > Its position, length, height, value, range, ...\
    > We are setting this part of the slider here.
    
   **📄PluginEditor.cpp**
   ``` c++
    Mu45effectAudioProcessorEditor::Mu45effectAudioProcessorEditor (Mu45effectAudioProcessor& p)
        : AudioProcessorEditor (&p), audioProcessor (p)
    {
        // plugin window size
        setSize (400, 300);

        // 1. create a pointer to the AudioParameters in the AudioProcessor
        // to connect the slider value and user parameter
        auto& processorParams = processor.getParameters();

        // 2. set the attributes of the slider
        mVolumeSlider.setBounds(0,0,80,280); // x, y, width, height (in pixels)
        mVolumeSlider.setSliderStyle(juce::Slider::SliderStyle::LinearVertical); // vertical slider
        mVolumeSlider.setTextBoxStyle(juce::Slider::TextBoxBelow, true, 80, 30); // readOnly, w, h

        // 3. set the range of the slider to the range of the AudioParameter
        // get the pointer to the parameter object
        juce::AudioParameterFloat* procParam = (juce::AudioParameterFloat*)processorParams.getUnchecked(0);
        mVolumeSlider.setRange(procParam->range.start, procParam->range.end);
        mVolumeSlider.setValue(*procParam);

        // 4. add the slider (It will now be a child of the AudioProcessorEditor object)
          addAndMakeVisible(mVolumeSlider);
    }
   ```    
> ### 🛠️ Build and Check time!
> Open the GUI and check if there is a slider. Does the GUI slider change the volume?\
> It should not change yet; we have not told the code to listen to the slider value change!

3. Make AudioPluginProcessorEditor inherit from the Slider::Listener class.

    **📄PluginEditor.h**
    ``` c++
    class Mu45effectAudioProcessorEditor  : public juce::AudioProcessorEditor, public juce::Slider::Listener
    {...}
    ```
4. Define a new public method `sliderValueChanged()`. we are *overriding* the method `Slider::Listener::sliderValueChanged`.

    **📄PluginEditor.h**
    ``` c++
    public:
        // Add this method to get called when a slider changes
        void sliderValueChanged(juce::Slider* slider) override;
    ```
5. Implement the `sliderValueChanged()` method.
    >A) Create a pointer to the list of AudioParameters in the PluginProcessor.\
    >B) Check to see *which* slider has been changed.\
    >C) Get a pointer to the AudioParameter we want to control with this slider\
    >D) Use the new value from the slider to set the AudioParameter\
    >E) We can use `DGB()` to print to the console while our plugin is running in debug mode.

    **📄PluginEditor.cpp**
    ``` c++
        // This method is called whenever any slider is changed
        void Mu45effectAudioProcessorEditor::sliderValueChanged(juce::Slider* slider)
        {
            // A) create a pointer to AudioParameters in the AudioProcessor
            auto& processorParams = processor.getParameters();

            // B) check to see which slider has been changed
            if (slider == &mVolumeSlider)
            {
                // C) get a pointer to the first parameter in the AudioProcessor    
                juce::AudioParameterFloat* procParam = 
                (juce::AudioParameterFloat*)processorParams.getUnchecked(0);

                // D) Use the value from the slider to set the AudioParamaeter in the AudioProcessor
                float sliderValue = mVolumeSlider.getValue();
                *procParam = sliderValue; // set the param

                // E) We can use DBG() for simple pring debugging
                DBG("Slider value changed: " << sliderValue);

                procParam->setValueNotifyingHost((float)mVolumeSlider.getValue());
            }
        }
    ```

6. Add a listener to the constructor.

   **📄PluginEditor.cpp**
   ``` c++
    Mu45effectAudioProcessorEditor::Mu45effectAudioProcessorEditor (Mu45effectAudioProcessor& p)
        : AudioProcessorEditor (&p), audioProcessor (p)
    {
        // plugin window size
        setSize (400, 300);

        // 1. create a pointer to the AudioParameters in the AudioProcessor
        // to connect the slider value and user parameter
        auto& processorParams = processor.getParameters();

        // 2. set the attributes of the slider
        mVolumeSlider.setBounds(0,0,80,280); // x, y, width, height (in pixels)
        mVolumeSlider.setSliderStyle(juce::Slider::SliderStyle::LinearVertical); // vertical slider
        mVolumeSlider.setTextBoxStyle(juce::Slider::TextBoxBelow, true, 80, 30); // readOnly, w, h

        // 3. set the range of the slider to the range of the AudioParameter
        // get the pointer to the parameter object
        juce::AudioParameterFloat* procParam = (juce::AudioParameterFloat*)processorParams.getUnchecked(0);
        mVolumeSlider.setRange(procParam->range.start, procParam->range.end);
        mVolumeSlider.setValue(*procParam);

        // 5. Set the ProcessorEditor to be a listener for our slider
        mVolumeSlider.addListener(this);

        // 4. add the slider (It will now be a child of the AudioProcessorEditor object)
          addAndMakeVisible(mVolumeSlider);
    }
   ```  

> ### 🛠️ Build and Check time!
> Now, the slider UI should change the volume!\
> How can we improve this plugin? Design choice? Aesthetics? User experience? ...

---
# Further Exploration
1. What if we need a parameter which value is not in a floating point?
    There are various classes JUCE is providing, including `AudioParameterFloat`!
    See https://docs.juce.com/master/classjuce_1_1AudioParameterFloat.html.
2. I want to stylize the slider differently.
    Refer to: https://docs.juce.com/master/classjuce_1_1Slider.html#af1caee82552143dd9ff0fc9f0cdc0888
3. I want to check out every other cool things JUCE have!
    JUCE Module document: https://docs.juce.com/master/index.html 
    ### Ask & tell us if the linked document is too overwhelming and you need help!

---
This is a rearranged document of the original handout made by Professor Luke Dahl.