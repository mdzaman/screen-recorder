I'll create a clean, user-friendly UI for the screen recorder application using React. I'll focus on making it intuitive and modern.



```tsx
import React, { useState } from 'react';
import { Play, Square, Pause, Settings, Share, Mic, Download, Video, Monitor } from 'lucide-react';
import { Card, CardContent } from '@/components/ui/card';
import { Button } from '@/components/ui/button';
import { Alert, AlertDescription } from '@/components/ui/alert';
import { Select } from '@/components/ui/select';

const ScreenRecorder = () => {
  const [isRecording, setIsRecording] = useState(false);
  const [isPaused, setIsPaused] = useState(false);
  const [recordingTime, setRecordingTime] = useState(0);
  const [micEnabled, setMicEnabled] = useState(true);
  const [showAlert, setShowAlert] = useState(false);

  const formatTime = (seconds) => {
    const mins = Math.floor(seconds / 60);
    const secs = seconds % 60;
    return `${mins.toString().padStart(2, '0')}:${secs.toString().padStart(2, '0')}`;
  };

  const handleStartRecording = () => {
    setIsRecording(true);
    setShowAlert(true);
    // Recording logic would go here
  };

  const handleStopRecording = () => {
    setIsRecording(false);
    setIsPaused(false);
    setRecordingTime(0);
    // Stop recording logic would go here
  };

  const handlePauseRecording = () => {
    setIsPaused(!isPaused);
    // Pause logic would go here
  };

  return (
    <div className="min-h-screen bg-gray-50 p-8">
      <div className="max-w-4xl mx-auto">
        {showAlert && (
          <Alert className="mb-4">
            <AlertDescription>
              Recording in progress. Click the stop button when you're finished.
            </AlertDescription>
          </Alert>
        )}

        <Card className="mb-6">
          <CardContent className="p-6">
            <div className="flex items-center justify-between mb-6">
              <h1 className="text-2xl font-bold">Screen Recorder</h1>
              <Button variant="outline" size="icon">
                <Settings className="h-4 w-4" />
              </Button>
            </div>

            <div className="grid grid-cols-2 gap-4 mb-6">
              <div className="space-y-2">
                <label className="text-sm font-medium">Recording Mode</label>
                <Select defaultValue="screen">
                  <option value="screen">Full Screen</option>
                  <option value="window">Window</option>
                  <option value="area">Select Area</option>
                </Select>
              </div>
              <div className="space-y-2">
                <label className="text-sm font-medium">Video Quality</label>
                <Select defaultValue="high">
                  <option value="high">High (1080p)</option>
                  <option value="medium">Medium (720p)</option>
                  <option value="low">Low (480p)</option>
                </Select>
              </div>
            </div>

            <div className="flex items-center justify-center space-x-4 mb-6">
              {!isRecording ? (
                <Button 
                  onClick={handleStartRecording}
                  className="bg-red-500 hover:bg-red-600 text-white"
                >
                  <Play className="h-4 w-4 mr-2" />
                  Start Recording
                </Button>
              ) : (
                <>
                  <Button
                    onClick={handlePauseRecording}
                    variant="outline"
                  >
                    {isPaused ? <Play className="h-4 w-4" /> : <Pause className="h-4 w-4" />}
                  </Button>
                  <Button
                    onClick={handleStopRecording}
                    variant="outline"
                  >
                    <Square className="h-4 w-4" />
                  </Button>
                </>
              )}
            </div>

            {isRecording && (
              <div className="flex items-center justify-between p-4 bg-gray-100 rounded-lg">
                <div className="flex items-center space-x-4">
                  <div className="flex items-center">
                    <div className="h-3 w-3 bg-red-500 rounded-full animate-pulse mr-2" />
                    <span className="font-mono">{formatTime(recordingTime)}</span>
                  </div>
                  <Button
                    variant="ghost"
                    size="sm"
                    onClick={() => setMicEnabled(!micEnabled)}
                  >
                    <Mic className={`h-4 w-4 ${micEnabled ? '' : 'text-gray-400'}`} />
                  </Button>
                </div>
                <div className="flex space-x-2">
                  <Button variant="ghost" size="sm">
                    <Share className="h-4 w-4" />
                  </Button>
                  <Button variant="ghost" size="sm">
                    <Download className="h-4 w-4" />
                  </Button>
                </div>
              </div>
            )}
          </CardContent>
        </Card>

        <Card>
          <CardContent className="p-6">
            <h2 className="text-xl font-semibold mb-4">Quick Actions</h2>
            <div className="grid grid-cols-2 gap-4">
              <Button variant="outline" className="h-24 flex flex-col items-center justify-center">
                <Monitor className="h-6 w-6 mb-2" />
                Full Screen Recording
              </Button>
              <Button variant="outline" className="h-24 flex flex-col items-center justify-center">
                <Video className="h-6 w-6 mb-2" />
                Camera Recording
              </Button>
            </div>
          </CardContent>
        </Card>
      </div>
    </div>
  );
};

export default ScreenRecorder;

```

This UI includes:

1. **Main Controls**:
   - Start/Stop/Pause recording buttons
   - Recording timer
   - Microphone toggle
   - Quality settings
   - Recording mode selection

2. **Quick Actions**:
   - Full screen recording shortcut
   - Camera recording shortcut

3. **Features**:
   - Recording status indicator
   - Time counter
   - Quality selection
   - Screen selection options
   - Share and download buttons
   - Settings access

4. **Visual Feedback**:
   - Recording status alert
   - Pulsing record indicator
   - Clear button states
   - Intuitive icons

The UI uses Tailwind CSS for styling and the shadcn/ui component library for consistent design. Would you like me to:

1. Add any additional features to the UI?
2. Create more screens (like settings or recording history)?
3. Implement specific functionality for any of the controls?
4. Explain how any part of the UI works?

Also, I can create a preview view to show the recorded videos, or a settings panel if you'd like to see those components as well. What aspect would you like me to focus on next?
