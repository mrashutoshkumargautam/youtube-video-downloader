import sys
from PyQt5.QtWidgets import QApplication, QVBoxLayout, QLabel, QLineEdit, QPushButton, QWidget, QProgressBar, QFileDialog, QMessageBox
from PyQt5.QtCore import QThread, pyqtSignal
from pytube import YouTube

class DownloadThread(QThread):
    progress = pyqtSignal(int)
    download_complete = pyqtSignal(str)
    error_occurred = pyqtSignal(str)

    def __init__(self, url, folder):
        super().__init__()
        self.url = url
        self.folder = folder

    def run(self):
        try:
            yt = YouTube(self.url, on_progress_callback=self.progress_function)
            stream = yt.streams.filter(progressive=True, file_extension='mp4').first()
            stream.download(output_path=self.folder)
            self.download_complete.emit("Download complete!")
        except Exception as e:
            self.error_occurred.emit(str(e))

    def progress_function(self, stream, chunk, bytes_remaining):
        total_size = stream.filesize
        bytes_downloaded = total_size - bytes_remaining
        percentage_of_completion = int(bytes_downloaded / total_size * 100)
        self.progress.emit(percentage_of_completion)


class YouTubeDownloader(QWidget):
    def __init__(self):
        super().__init__()

        # Set up the GUI layout
        self.setWindowTitle('YouTube Video Downloader')
        self.setGeometry(400, 150, 600, 250)

        layout = QVBoxLayout()

        # Label
        self.label = QLabel("Enter YouTube Video URL:")
        self.label.setStyleSheet("font-size: 16px;")
        layout.addWidget(self.label)

        # URL input field
        self.url_input = QLineEdit()
        self.url_input.setStyleSheet("font-size: 16px; height: 30px;")
        layout.addWidget(self.url_input)

        # Download button
        self.download_button = QPushButton('Download')
        self.download_button.setStyleSheet("font-size: 16px; height: 40px;")
        self.download_button.clicked.connect(self.start_download)
        layout.addWidget(self.download_button)

        # Progress bar
        self.progress_bar = QProgressBar()
        self.progress_bar.setStyleSheet("height: 30px;")
        self.progress_bar.setValue(0)
        layout.addWidget(self.progress_bar)

        # Set layout
        self.setLayout(layout)

    def start_download(self):
        url = self.url_input.text()

        if not url.strip():
            QMessageBox.warning(self, "Error", "Please enter a valid YouTube URL.")
            return

        # Ask for download location
        folder = QFileDialog.getExistingDirectory(self, "Select Download Directory")
        if not folder:
            QMessageBox.warning(self, "Error", "Please select a valid directory.")
            return

        # Start the download thread
        self.download_thread = DownloadThread(url, folder)
        self.download_thread.progress.connect(self.update_progress)
        self.download_thread.download_complete.connect(self.on_download_complete)
        self.download_thread.error_occurred.connect(self.on_error)
        self.download_thread.start()

    def update_progress(self, progress):
        self.progress_bar.setValue(progress)

    def on_download_complete(self, message):
        QMessageBox.information(self, "Success", message)

    def on_error(self, error_message):
        QMessageBox.critical(self, "Error", f"An error occurred: {error_message}")


# Main application loop
if __name__ == '__main__':
    app = QApplication(sys.argv)
    window = YouTubeDownloader()
    window.show()
    sys.exit(app.exec_())
