# squint-lens — лупа по прищуру

Идея: когда я наклоняюсь к экрану и щурюсь, показывать лупу
в той области, куда смотрю.

## Этапы
1. [x] Проверить камеру ноутбука
   - встроенная камера — индекс `1` (`0` — виртуальная iVCam с телефона)
   - в блокноте: `CAMERA = 1`, открывать через `cv2.VideoCapture(CAMERA, cv2.CAP_DSHOW)`
   - цикл с окном — в `try/finally` (release + destroyAllWindows), закрытие по Esc или крестику
2. [ ] MediaPipe Face Landmarker: видит ли лицо под моим углом
   - [x] новый Tasks API работает (`mp.solutions` удалён): `mp.tasks.vision.FaceLandmarker`,
         режим `VIDEO`, `detect_for_video(mp_image, ts)`, 478 точек (с зрачками)
   - [x] точки рисуются на лице
   - [ ] проверить в рабочих условиях (рисовать только веки + зрачки, надпись NO FACE):
     - [ ] сижу как обычно за монитором (камера сбоку) — точки держатся
     - [ ] наклон к экрану + прищур — точки на веках не слетают
     - [ ] в очках
     - [ ] вечером при обычном свете
     - [ ] голова повёрнута к монитору — лицо не теряется
   - точки глаз: веки/уголки `159, 145, 33, 133` и `386, 374, 362, 263`, зрачки `468–477`
3. [ ] Признаки: открытость глаз, размер лица в кадре (наклон)
   - открытость глаза: расстояние верх–низ века (159–145, 386–374),
     делённое на ширину глаза (33–133, 362–263) — чтобы не зависело от расстояния до камеры
   - наклон к экрану: размер лица в кадре растёт
4. [ ] Записать свой датасет: обычно / наклон + прищур, днём и вечером, в очках
5. [ ] Модель на PyTorch: «рассматриваю / не рассматриваю»
6. [ ] Лупа вокруг курсора по сигналу модели (Windows Magnification API)
7. [ ] Позже: калибровка взгляда, лупа туда, куда смотрю

## Условия
- камера ноутбука, стоит сбоку от монитора
- ничего не покупаем
- всё работает локально, видео никуда не отправляется

#### Окружение
```
python -m venv .venv
.venv\Scripts\activate
pip install opencv-python mediapipe ipykernel
python -c "import mediapipe, cv2; print(mediapipe.__version__, cv2.__version__)"
deactivate
```
Модель Face Landmarker (один раз, ~4 МБ, из папки 05_squint_lens):
```
mkdir models
Invoke-WebRequest "https://storage.googleapis.com/mediapipe-models/face_landmarker/face_landmarker/float16/latest/face_landmarker.task" -OutFile models\face_landmarker.task
```
В коде путь: `models/face_landmarker.task`.

Внимание: в .venv стоят и `opencv-python`, и `opencv-contrib-python` (второй тянет mediapipe).
Пока работает — не трогать; если в `cv2` пойдут странные ошибки — оставить только contrib.

Добавление пути для окружения

палитра → Python: Select Interpreter → Enter interpreter path... →  C:\Users\excavator\projects\cv\05_squint_lens\.venv\Scripts\python.exe
