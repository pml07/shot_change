### Histogram
> 原理：將影像由 (r, g, b) 轉為灰階影像，計算影片中，第`i`個frame與第`i+1`個frame的bins之間的差距比例，總共256個 bins
---
* 實作 & 程式碼講解

1. 影像轉成灰階
```cpp=
def load_images_from_folder(folder):
  images=[]
  if folder == ngc_dir:
    filenames = [img for img in glob.glob(folder + "/*.jpeg")]
  else:
    filenames = [img for img in glob.glob(folder + "/*.jpg")]
  filenames.sort()
  for file in filenames:
    img = cv2.imread(file)
    img_gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY) 
    if img is not None:
      images.append(img_gray)
  return images
```
* Output
![](https://i.imgur.com/Uft2DXe.png)


2. 計算每個frame的bins分布
```cpp=
news_img = load_images_from_folder(news_dir)
img = cv2_imshow(news_img[90])

hist = cv2.calcHist([news_img[90]], [0], None, [256], [0, 255])
plt.figure()
plt.title("Grayscale Histogram")
plt.xlabel("Bins")
plt.ylabel("# of Pixels")
plt.plot(hist)
plt.xlim([0, 255])
```
* news影片中某個frame的bins分佈之折線圖
![](https://i.imgur.com/t8qI6C5.png)


3. 比較第`i`個frame 與 第`i+1`個frame，計算bins差值
```cpp=
def detect(self):
    for index in range(0, len(self.img)-1):
      hist = cv2.calcHist([self.img[index]], [0], None, [256], [0,255])
      hist_next = cv2.calcHist([self.img[index+1]], [0], None, [256], [0,255])
      diff = 0
      hist_sum = 0
      for hist_index, hist_value in enumerate(hist):
        hist_sum += hist_value[0]
        diff += abs(hist_next[hist_index][0] - hist_value[0])
      self.result.append(diff/hist_sum)
```

4. 設定threshold，選出bins差值高於threshold的frame
```cpp=
def shot_change_frames(self):
    for i,v in enumerate(self.result):
        if v > self.threshold:
          self.shot_frames.append(i+1)
    return self.shot_frames
```
* Result
![](https://i.imgur.com/qeGDLd4.png)

### Edge detection
> 原理：使用 Edge change ratio來偵測 shot change
---
* 實作 & 程式講解
1. 影像轉成灰階(同Histogram)
2. 計算第`i`個frame 與 第`i+1`個frame的ECR
```cpp=
def detect(self):
    dilate_rate = 5
    high_edge_threshold = 200
    safe_div = lambda x,y: 0 if y == 0 else x / y
    for index in range(0, len(self.img)-1):
      edge = cv2.Canny(self.img[index], self.low_edge_threshold, high_edge_threshold)
      dilated = cv2.dilate(edge, np.ones((dilate_rate, dilate_rate)))
      inverted = (255 - dilated)
      edge2 = cv2.Canny(self.img[index+1], self.low_edge_threshold, high_edge_threshold)
      dilated2 = cv2.dilate(edge2, np.ones((dilate_rate, dilate_rate)))
      inverted2 = (255 - dilated2)
      log_and1 = (edge2 & inverted)
      log_and2 = (edge & inverted2)
    
