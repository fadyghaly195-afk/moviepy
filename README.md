from moviepy.editor import *
from moviepy.video.fx.all import resize, lum_contrast, fadein, fadeout, colorx, crop
import numpy as np

# مسارات الملفات
logo_path = '/mnt/data/A_collection_of_ten_Arabic_logo_designs_for_"حكاية.png'
audio_bg_path = '/mnt/data/mystery_background.mp3'   # موسيقى خلفية تصاعدية
audio_sfx_path = '/mnt/data/sfx_chime.mp3'           # صوت قصير عند ظهور النصوص
output_path = '/mnt/data/hkaya_cinematic_max.mp4'

# إعداد النصوص: (النص، وقت البداية، وقت النهاية، حجم الخط)
texts = [
    ("حكاية مش كاملة…", 0, 2, 70),
    ("لكل حكاية تركاية تخص صاحبها", 2, 5, 55),
    ("معاكم فادي غالي في حكاية مش كاملة", 5, 8, 50),
    ("كل لحظة تحكي قصة…", 8, 11, 45),
    ("تابعونا لمعرفة النهاية", 11, 14, 40)
]

# خلفية سوداء سينمائية + Color Grading خفيف
bg = ColorClip(size=(720,1280), color=(10,10,10), duration=15)
bg = bg.fx(colorx, 1.2)  # تعزيز التباين والألوان

# شعار مع Flicker + Glow + Lens Flare ديناميكي
logo = ImageClip(logo_path).set_duration(15).resize(width=400).set_pos('center')
def flicker_glow(img, t):
    lum = 20 + 15*np.sin(4*t)
    contrast = 20
    img = lum_contrast(img, lum=lum, contrast=contrast)
    # Lens Flare متحرك
    h, w, _ = img.shape
    y, x = np.ogrid[:h, :w]
    mask = ((y-h/2)**2 + (x-w/2)**2) < (h/4)**2
    img[mask] = np.clip(img[mask] + 25*np.sin(t*2), 0, 255)
    return img
logo = logo.fl_image(lambda img, t: flicker_glow(img, t))

clips = [bg, logo]

# إضافة النصوص السينمائية مع Glow وLight Rays
for txt, start, end, size in texts:
    txt_clip = TextClip(txt, fontsize=size, color='white', font='Arial', method='caption')
    txt_clip = txt_clip.set_start(start).set_end(end)

    # Fade in + Zoom خفيف
    txt_clip = txt_clip.resize(lambda t: 1 + 0.1*t)
    txt_clip = txt_clip.crossfadein(0.5)

    # Light Rays وSlide حسب النص
    if start == 2:
        txt_clip = txt_clip.set_pos(lambda t: ('center', 1280 - 700*(t-start)))
        txt_clip = txt_clip.fl_image(lambda img: lum_contrast(img, lum=15, contrast=15))
    elif start >= 5:
        txt_clip = txt_clip.set_pos(('center', 1000))
        txt_clip = txt_clip.fl_image(lambda img: lum_contrast(img, lum=30, contrast=45))
    else:
        txt_clip = txt_clip.set_pos('center')

    # صوت قصير عند ظهور النص
    sfx = AudioFileClip(audio_sfx_path).subclip(0,0.5).volumex(0.7).set_start(start)
    txt_clip = txt_clip.set_audio(sfx)

    clips.append(txt_clip)

# Simulated Camera Pan خفيف لزيادة الديناميكية
def pan_clip(get_frame, t):
    frame = get_frame(t)
    h, w, _ = frame.shape
    # تحريك بسيط في الاتجاه الرأسي
    y_shift = int(20*np.sin(t*1.5))
    return frame[y_shift:h, :, :]
final_bg = CompositeVideoClip(clips, size=(720,1280)).fl(pan_clip)

# إضافة الموسيقى الخلفية مع تصاعد صوتي
audio_bg = AudioFileClip(audio_bg_path).subclip(0,15).volumex(lambda t: 0.4 + 0.6*(t/15))
final = final_bg.set_audio(audio_bg)

# تصدير النسخة السينمائية القصوى
final.write_videofile(output_path, fps=24)
