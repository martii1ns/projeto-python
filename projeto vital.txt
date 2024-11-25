import pytubefix
class VideoAnalyzer:
 def __init__(self):
 print("Analisador inicializado.")
 def get_video_metadata(self, url):
 try:
 yt = pytubefix.YouTube(url)
 metadata = {
 "Título": yt.title,
 "Duração (segundos)": yt.length,
 "Visualizações": yt.views,
 "Autor": yt.author,
 "Descrição": yt.description,
 }
 print("\nMetadados do vídeo:")
 for key, value in metadata.items():
 print(f"{key}: {value}")
 return metadata, yt
 except Exception as e:
 print(f"Erro ao obter metadados: {e}")
 return None, None
 def list_available_streams(self, yt):
 try:
 print("\nQualidades disponíveis:")
 video_streams = yt.streams.filter(progressive=False,
file_extension='mp4').order_by("resolution").desc()
 audio_streams = yt.streams.filter(only_audio=True,
file_extension='mp4').order_by("abr").desc()
 best_streams = {}
 for stream in video_streams:
 resolution = stream.resolution
 codec = stream.video_codec
 if resolution not in best_streams:
 best_streams[resolution] = {"video": stream, "audio": None}
 else:
 current_best = best_streams[resolution]["video"]
 if (codec == "vp9" and current_best.video_codec != "vp9") or
(codec == "av01.0.09M.08" and current_best.video_codec != "av01.0.09M.08"):
 best_streams[resolution]["video"] = stream
 for audio in audio_streams:
 best_resolution = None
 for resolution, streams in best_streams.items():
 if streams["audio"] is None or audio.abr >
streams["audio"].abr:
 best_streams[resolution]["audio"] = audio
 for resolution, streams in best_streams.items():
 video = streams["video"]
 audio = streams["audio"]
 print(f"- Resolução: {resolution}, Codec de Vídeo:
{video.video_codec}, "
 f"Codec de Áudio: {audio.audio_codec if audio else 'N/A'}, "
 f"Taxa de bits de Áudio: {audio.abr if audio else 'N/A'}, "
 f"Tipo: 'Vídeo e Áudio Separado'")
 return best_streams, video_streams, audio_streams
 except Exception as e:
 print(f"Erro ao listar streams: {e}")
 return None, None, None
 def prompt_for_valid_url(self):
 while True:
 video_url = input("Por favor, insira a URL do vídeo do YouTube:
").strip()

 if video_url.startswith("https://www.youtube.com/watch") or
video_url.startswith("https://youtu.be"):
 return video_url
 else:
 print("URL inválida! Por favor, insira uma URL válida do YouTube.")
 def download_video(self, yt, video_streams, audio_streams):
 separate_choice = input("Deseja baixar o vídeo separado do áudio? Y para
sim, N para não: ").strip().upper()
 if separate_choice == 'Y':
 print("\nVocê escolheu baixar o vídeo e o áudio separadamente.")
 else:
 print("\nVocê escolheu baixar o vídeo e o áudio juntos.")
 print("\nEscolha a resolução do vídeo que você deseja baixar:")
 for i, stream in enumerate(video_streams):
 print(f"{i+1} - {stream.resolution}")
 video_choice = int(input("Escolha o número da resolução do vídeo:
").strip())
 if video_choice < 1 or video_choice > len(video_streams):
 print("Escolha inválida. Baixando a maior qualidade disponível.")
 video_choice = 1
 chosen_video = video_streams[video_choice - 1]
 print("\nEscolha a taxa de bits de áudio que você deseja baixar:")
 for i, stream in enumerate(audio_streams):
 print(f"{i+1} - {stream.abr} kbps")
 audio_choice = int(input("Escolha o número da taxa de bits de áudio:
").strip())
 if audio_choice < 1 or audio_choice > len(audio_streams):
 print("Escolha inválida. Baixando o áudio com maior taxa de bits
disponível.")
 audio_choice = 1
 chosen_audio = audio_streams[audio_choice - 1]
 if separate_choice == 'Y':
 print(f"\nBaixando vídeo na resolução {chosen_video.resolution}...")
 chosen_video.download(output_path='.', filename='video.mp4')
 print(f"\nBaixando áudio de {chosen_audio.abr} kbps...")
 chosen_audio.download(output_path='.', filename='audio.mp4')
 print("Download completo! Vídeo e áudio foram baixados separadamente.")
 else:
 print(f"\nBaixando o vídeo com o áudio junto na melhor qualidade
disponível...")
 best_stream = yt.streams.filter(progressive=True,
file_extension='mp4').order_by("resolution").desc().first() # type: ignore
 best_stream.download(output_path='.', filename='video_com_audio.mp4')
 print("Download completo! Vídeo com áudio foi baixado junto.")
if __name__ == "__main__":
 analyzer = VideoAnalyzer()

 video_url = analyzer.prompt_for_valid_url()

 metadata, yt = analyzer.get_video_metadata(video_url)
 if yt is None:
 print("Erro ao obter o vídeo, encerrando.")
 exit()
 best_streams, video_streams, audio_streams =
analyzer.list_available_streams(yt)
 download_choice = input("\nVocê deseja baixar o vídeo? Y para sim, N para não:
").strip().upper()
 if download_choice == 'Y':
 analyzer.download_video(yt, video_streams, audio_streams)
 else:
 print("Você escolheu não baixar o vídeo.")