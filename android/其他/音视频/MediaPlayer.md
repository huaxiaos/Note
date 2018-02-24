# 禁止其他app的音乐播放

```
private final AudioManager.OnAudioFocusChangeListener mOnAudioFocusChangeListener = new AudioManager.OnAudioFocusChangeListener() {
        @Override
        public void onAudioFocusChange(int focusChange) {

        }
    };

public boolean muteAudio(boolean bMute) {
    boolean isSuccess = false;
    AudioManager am = (AudioManager) MediaApplication.getContext().getSystemService(AUDIO_SERVICE);
    if (am == null) return false;

    if (bMute) {
        int result = am.requestAudioFocus(mOnAudioFocusChangeListener,
                AudioManager.STREAM_MUSIC,
                AudioManager.AUDIOFOCUS_GAIN_TRANSIENT);
        isSuccess = (result == AudioManager.AUDIOFOCUS_REQUEST_GRANTED);
    } else {
        int result = am.abandonAudioFocus(null);
        isSuccess = (result == AudioManager.AUDIOFOCUS_REQUEST_GRANTED);
    }

    return isSuccess;
}
```