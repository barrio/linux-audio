#include <iostream>
#include <fstream>
#include <vector>
#include <alsa/asoundlib.h>

// Funktion zum Lesen der WAV-Datei
std::vector<char> read_wav_file(const std::string& filename) {
    std::ifstream file(filename, std::ios::binary);
    if (!file) {
        throw std::runtime_error("Konnte die WAV-Datei nicht öffnen.");
    }

    // Lesen Sie die Datei in einen Puffer
    std::vector<char> buffer((std::istreambuf_iterator<char>(file)), std::istreambuf_iterator<char>());
    return buffer;
}

// Funktion zum Abspielen der WAV-Datei mit ALSA
void play_wav(const std::vector<char>& buffer) {
    snd_pcm_t *pcm_handle;
    snd_pcm_hw_params_t *params;
    int rc;
    unsigned int sample_rate;
    int channels;
    snd_pcm_uframes_t frames;

    // Öffnen des PCM-Geräts für die Wiedergabe
    rc = snd_pcm_open(&pcm_handle, "default", SND_PCM_STREAM_PLAYBACK, 0);
    if (rc < 0) {
        throw std::runtime_error("Konnte PCM-Gerät nicht öffnen: " + std::string(snd_strerror(rc)));
    }

    // Hardware-Parameter-Objekt initialisieren
    snd_pcm_hw_params_alloca(&params);
    snd_pcm_hw_params_any(pcm_handle, params);

    // Hardware-Parameter einstellen
    snd_pcm_hw_params_set_access(pcm_handle, params, SND_PCM_ACCESS_RW_INTERLEAVED);
    snd_pcm_hw_params_set_format(pcm_handle, params, SND_PCM_FORMAT_S16_LE);

    // Abrufen der Abtastrate und der Kanäle aus der WAV-Datei
    sample_rate = *reinterpret_cast<const unsigned int*>(&buffer[24]);
    channels = *reinterpret_cast<const unsigned short*>(&buffer[22]);

    snd_pcm_hw_params_set_rate_near(pcm_handle, params, &sample_rate, nullptr);
    snd_pcm_hw_params_set_channels(pcm_handle, params, channels);

    // Hardware-Parameter auf das PCM-Gerät anwenden
    rc = snd_pcm_hw_params(pcm_handle, params);
    if (rc < 0) {
        throw std::runtime_error("Konnte Hardware-Parameter nicht einstellen: " + std::string(snd_strerror(rc)));
    }

    // Anzahl der Frames pro Puffer abrufen
    snd_pcm_hw_params_get_period_size(params, &frames, nullptr);

    // Abspielen der WAV-Datei
    size_t data_offset = 44; // Der Datenbereich beginnt bei Byte 44 in einer typischen WAV-Datei
    size_t data_size = buffer.size() - data_offset;
    const char* data = buffer.data() + data_offset;

    while (data_size > 0) {
        rc = snd_pcm_writei(pcm_handle, data, frames);
        if (rc == -EPIPE) {
            // Buffer underrun
            snd_pcm_prepare(pcm_handle);
        } else if (rc < 0) {
            std::cerr << "Fehler beim Schreiben der Audiodaten: " << snd_strerror(rc) << std::endl;
        } else {
            data += rc * channels * 2; // 2 bytes per sample for S16_LE
            data_size -= rc * channels * 2;
        }
    }

    // PCM-Gerät schließen
    snd_pcm_drain(pcm_handle);
    snd_pcm_close(pcm_handle);
}

int main(int argc, char* argv[]) {
    if (argc < 2) {
        std::cerr << "Verwendung: " << argv[0] << " <WAV-Datei>" << std::endl;
        return 1;
    }

    try {
        std::vector<char> wav_data = read_wav_file(argv[1]);
        play_wav(wav_data);
    } catch (const std::exception& e) {
        std::cerr << "Fehler: " << e.what() << std::endl;
        return 1;
    }

    return 0;
}
