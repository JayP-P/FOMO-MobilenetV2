import cv2
import numpy as np
import tensorflow as tf
import time

# --- CONFIGURAÇÕES ---
MODEL_PATH = r'Caminho para o modelo .tflite'
LABELS_PATH = 'Caminho para as classes.txt'
DETECTION_THRESHOLD = 0.5

# --- MELHORIA: Margens independentes e mais claras ---
MARGIN_HORIZONTAL = 0.1 # Margem para esquerda/direita
MARGIN_VERTICAL = 0.05    # Margem para topo/base

# --- ALTERAÇÃO PRINCIPAL: Configuração da Câmera IP ---
# Coloque a URL do stream de vídeo da sua câmera IP aqui.
# Se esta variável for uma string vazia "", o código usará a webcam local (índice 0).
#
# Exemplos de formatos de URL:
# - RTSP: "rtsp://usuario:senha@192.168.1.10:554/stream1"
# - HTTP: "http://usuario:senha@192.168.1.10/videostream.cgi"
IP_CAMERA_URL = "" # <--- COLOQUE A URL DA SUA CÂMERA AQUI

# --- CARREGAR MODELO E LEGENDAS ---
try:
    interpreter = tf.lite.Interpreter(model_path=MODEL_PATH)
    interpreter.allocate_tensors()
except Exception as e:
    print(f"Erro: Não foi possível carregar o modelo TFLite. Verifique o caminho.")
    print(f"Detalhe do erro: {e}")
    exit()

input_details = interpreter.get_input_details()
output_details = interpreter.get_output_details()

height = input_details[0]['shape'][1]
width = input_details[0]['shape'][2]
INPUT_SIZE = (width, height)

print(f"Modelo carregado. Tamanho de entrada esperado: {INPUT_SIZE}")

try:
    with open(LABELS_PATH, 'r') as f:
        labels = [line.strip() for line in f.readlines()]
except FileNotFoundError:
    print(f"Erro: Arquivo de legendas '{LABELS_PATH}' não encontrado.")
    exit()

# --- ACESSAR A FONTE DE VÍDEO (Câmera IP ou Webcam) ---
if IP_CAMERA_URL != "":
    video_source = IP_CAMERA_URL
    print(f"Conectando à câmera IP: {video_source}")
else:
    video_source = 0 # Webcam padrão
    print("Usando a webcam local.")

cap = cv2.VideoCapture(video_source)

if not cap.isOpened():
    print(f"Erro: Não foi possível acessar a fonte de vídeo: {video_source}")
    exit()

print("\nPressione 'q' para sair.")

# --- LOOP PRINCIPAL DE PROCESSAMENTO DE VÍDEO ---
while True:
    ret, frame = cap.read()
    if not ret:
        print("Erro: Não foi possível ler o frame. A conexão pode ter sido perdida. Encerrando...")
        break

    image_for_drawing = frame.copy()
    original_h, original_w, _ = image_for_drawing.shape

    # Pré-processamento da imagem para o modelo
    input_image = cv2.resize(frame, INPUT_SIZE)
    input_image = cv2.cvtColor(input_image, cv2.COLOR_BGR2RGB)

    if input_details[0]['dtype'] == np.uint8:
        input_data = np.expand_dims(input_image, axis=0)
    else:
        input_data = np.expand_dims(input_image, axis=0).astype(np.float32) / 255.0

    # --- EXECUÇÃO DA INFERÊNCIA ---
    start_time = time.time()
    interpreter.set_tensor(input_details[0]['index'], input_data)
    interpreter.invoke()
    output_data = interpreter.get_tensor(output_details[0]['index'])
    end_time = time.time()

    # --- PÓS-PROCESSAMENTO ---
    output_grid = output_data[0]
    grid_h, grid_w, num_classes = output_grid.shape
    
    cell_pixel_w = original_w / grid_w
    cell_pixel_h = original_h / grid_h
    detection_radius = int(min(cell_pixel_w, cell_pixel_h) * 0.75)
    if detection_radius < 5: detection_radius = 5
    
    detection_count = 0

    margin_x_start = int(grid_w * MARGIN_HORIZONTAL)
    margin_x_end = int(grid_w * (1 - MARGIN_HORIZONTAL))
    margin_y_start = int(grid_h * MARGIN_VERTICAL)
    margin_y_end = int(grid_h * (1 - MARGIN_VERTICAL))

    pt1 = (int(margin_x_start * cell_pixel_w), int(margin_y_start * cell_pixel_h))
    pt2 = (int(margin_x_end * cell_pixel_w), int(margin_y_end * cell_pixel_h))
    cv2.rectangle(image_for_drawing, pt1, pt2, (255, 0, 0), 2)

    for y in range(grid_h):
        for x in range(grid_w):
            if not (margin_x_start <= x < margin_x_end and margin_y_start <= y < margin_y_end):
                continue

            scores = output_grid[y][x]
            class_id = np.argmax(scores)
            confidence = scores[class_id]
            
            is_background = (len(labels) == num_classes and class_id == 0) or \
                            (len(labels) < num_classes and labels[class_id].lower() in ['background', 'uncertain'])

            if confidence > DETECTION_THRESHOLD and not is_background:
                detection_count += 1
                
                center_x = int((x + 0.5) * cell_pixel_w)
                center_y = int((y + 0.5) * cell_pixel_h)
                
                cv2.circle(image_for_drawing, (center_x, center_y), detection_radius, (0, 255, 0), 2)
                
                label_text = f"{labels[class_id]}: {confidence:.2f}"
                cv2.putText(image_for_drawing, label_text, (center_x - detection_radius, center_y - detection_radius - 10), 
                            cv2.FONT_HERSHEY_SIMPLEX, 0.5, (0, 255, 0), 1)

    # Adiciona o texto de status
    inference_time = end_time - start_time
    fps = 1 / inference_time if inference_time > 0 else 0
    status_text = f"Objetos: {detection_count} | Inferencia: {inference_time * 1000:.1f}ms | FPS: {fps:.1f}"

    cv2.putText(image_for_drawing, status_text, (10, original_h - 10),
                cv2.FONT_HERSHEY_SIMPLEX, 0.6, (255, 255, 255), 2, cv2.LINE_AA)
    
    cv2.imshow('Deteccao de Objetos - Camera IP (FOMO)', image_for_drawing)

    if cv2.waitKey(1) & 0xFF == ord('q'):
        break

# --- LIMPEZA AO SAIR ---
cap.release()
cv2.destroyAllWindows()
