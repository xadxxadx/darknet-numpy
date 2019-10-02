# darknet-numpy
Make numpy type as input of darknet library
## 1. The basic method

a. in image.h of darknet project, Add code below
image ndarray_to_image(unsigned char* src, long* shape, long* strides)
{
	int h = shape[0];
	int w = shape[1];
	int c = shape[2];
	int step_h = strides[0];
	int step_w = strides[1];
	int step_c = strides[2];
	image im = make_image(w, h, c);
	int i, j, k;
	int index1, index2 = 0;

	for (i = 0; i < h; ++i) {
		for (k = 0; k < c; ++k) {
			for (j = 0; j < w; ++j) {
				index1 = k*w*h + i*w + j;
				index2 = step_h*i + step_w*j + step_c*k;
				// fprintf(stderr, "w=%d h=%d c=%d step_w=%d step_h=%d step_c=%d \n", w, h, c, step_w, step_h, step_c); 
				// fprintf(stderr, "im.data[%d]=%u data[%d]=%f \n", index1, src[index2], index2, src[index2]/255.); 
				im.data[index1] = src[index2] / 255.;
			}
		}
	}
	rgbgr_image(im);

	return im;
  
b. in image.h of darknet project, Add code below
image ndarray_to_image(unsigned char* src, long* shape, long* strides);

c. rebuild

d. in darknet.py, Add code below
ndarray_image = lib.ndarray_to_image
ndarray_image.argtypes = [POINTER(c_ubyte), POINTER(c_long), POINTER(c_long)]
ndarray_image.restype = IMAGE
def nparray_to_image(img):
    data = img.ctypes.data_as(POINTER(c_ubyte))
    image = ndarray_image(data, img.ctypes.shape, img.ctypes.strides)
    return image

e. finially, you can replace 
im = load_image(image.encode('utf-8'), 0, 0) 
by
image = cv2.imread(img_path)
image = np.array(image)
im = dn.nparray_to_image(image)

## 2. If you occur "expected LP_c_long instance instead of c_longlong_Array_3"
It's a problem cause by specific numpy version.
It means that numpy.ctypes.shape return type of c_longlongArray_3, but API parameters is type of long pointer.
My solution is ...

a. Change API in image.c/image.h like this below
image ndarray_to_image(unsigned char* src, long long* shape, long long* strides)

b. after rebuild, in darknet.py, like step 'd', but we need to fit parameters type because of numpy version
ndarray_image = lib.ndarray_to_image
ndarray_image.argtypes = [POINTER(c_ubyte), ARRAY(c_longlong,3), ARRAY(c_longlong,3)]
ndarray_image.restype = IMAGE
def nparray_to_image(img):
    data = img.ctypes.data_as(POINTER(c_ubyte))
    image = ndarray_image(data, img.ctypes.shape, img.ctypes.strides)
    return image
    
c. So, you can pass numpy data to darknet api right now

