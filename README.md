# Practica2_Camara
Objetivo. Visualizar un Preview en pantalla que muestre lo que la cámara registre, capturaremos unas fotografías y las almacenaremos en un sitio relativo a la aplicación.

En app/build.gradle vamos a instalar la dependencia de CameraX y habilitar ViewBinding. android { viewBinding { enabled = true } } implementation ("androidx.camera:camera-core:1.2.3") implementation ("androidx.camera:camera-camera2:1.2.3") implementation ("androidx.camera:camera-lifecycle:1.2.3") implementation ("androidx.lifecycle:lifecycle-runtime-ktx:2.6.1") implementation ("androidx.concurrent:concurrent-futures-ktx:1.1.0")
Utilizamos los siguientes permisos en el AndroidManifest.xml
Registramos esta Activity (posteriormente la crearemos)
4. En activity_main, dejamos un simple botón
5. En MainActivity, pediremos permisos para acceder a la cámara para poder cambiarnos a la Activity de la cámara.
class MainActivity : AppCompatActivity() { private val PERMISSION_ID = 34 private lateinit var binding: ActivityMainBinding override fun onCreate(savedInstanceState: Bundle?) { super.onCreate(savedInstanceState) binding = ActivityMainBinding.inflate(layoutInflater) setContentView(binding.root) binding.btnOpenCamera.setOnClickListener { if(checkCameraPermission()){ openCamera() } else{ requestPermissions() } }

} private fun openCamera(){ val intent = Intent(this, CameraActivity::class.java) startActivity(intent) } @SuppressLint("MissingSuperCall") override fun onRequestPermissionsResult(requestCode: Int, permissions: Array, grantResults: IntArray) { if (requestCode == PERMISSION_ID) { if ((grantResults.isNotEmpty() && grantResults[0] == PackageManager.PERMISSION_GRANTED)) { openCamera() } else { Toast.makeText(this,"Aun requieres permiso", Toast.LENGTH_SHORT).show() } } } private fun checkCameraPermission(): Boolean{ return ActivityCompat

.checkSelfPermission(this, Manifest.permission.CAMERA)== PackageManager.PERMISSION_GRANTED }

private fun requestPermissions() { ActivityCompat.requestPermissions( this, arrayOf(Manifest.permission.CAMERA, Manifest.permission.ACCESS_FINE_LOCATION), PERMISSION_ID ) } } 6. Creamos el layout para nuestra Actividad de Cámara, esta consta de un TextureView donde desplegaremos el Preview de la cámara, y el botón para hacer la captura:

<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android" xmlns:app="http://schemas.android.com/apk/res-auto" android:orientation="horizontal" android:layout_width="match_parent" android:layout_height="match_parent"

<androidx.camera.view.PreviewView android:id="@+id/camera_preview" android:layout_width="0dp" android:layout_height="0dp" android:layout_weight="1" app:layout_constraintBottom_toBottomOf="parent" app:layout_constraintEnd_toEndOf="parent" app:layout_constraintStart_toStartOf="parent" app:layout_constraintTop_toTopOf="parent" />

</androidx.constraintlayout.widget.ConstraintLayout>

Creamos nuestra clase CameraActivity class CameraActivity : AppCompatActivity(){ private lateinit var binding: ActivityCameraBinding private var imageCapture: ImageCapture? = null
override fun onCreate(savedInstanceState: Bundle?) { super.onCreate(savedInstanceState)

binding = ActivityCameraBinding.inflate(layoutInflater) setContentView(binding.root)

lifecycleScope.launch { startCamera() }

binding.captureButton.setOnClickListener { takePhoto() } }

private suspend fun startCamera(){ val cameraProvider = ProcessCameraProvider.getInstance(this).await()

// Construimos el preview (aquí podemos hacer configuraciones) val preview = Preview.Builder() .build() .also { it.setSurfaceProvider(binding.cameraPreview.surfaceProvider) } // Seleccionamos la cámara trasera val cameraSelector = CameraSelector.DEFAULT_BACK_CAMERA try { // Atamos nuestra cámara al ciclo de vida de nuestro Activity cameraProvider.run { unbindAll() imageCapture = ImageCapture.Builder().build() bindToLifecycle(this@CameraActivity, cameraSelector, preview, imageCapture) }

} catch(exc: Exception) { Toast.makeText(this, "No se pudo hacer bind al lifecycle", Toast.LENGTH_SHORT).show() } } fun takePhoto() { // Creamos un nombre único para cada foto val format = SimpleDateFormat("dd-MM-yyyyy-HH:mm:ss:SSS", Locale.US) .format(System.currentTimeMillis()) val name = "beduPhoto $format"

val contentValues = ContentValues().apply { put(MediaStore.MediaColumns.DISPLAY_NAME, name) put(MediaStore.MediaColumns.MIME_TYPE, "image/jpeg") if(Build.VERSION.SDK_INT > Build.VERSION_CODES.P) { put(MediaStore.Images.Media.RELATIVE_PATH, "Pictures/CameraX-Image") // La carpeta donde se guarda } }

// Creamos el builder para la configuración del archivo y los metadatos val outputOptions = ImageCapture.OutputFileOptions .Builder( contentResolver, MediaStore.Images.Media.EXTERNAL_CONTENT_URI, contentValues) .build()

// Seteamos el listener de cuando la captura sea efectuada imageCapture?.takePicture( outputOptions, ContextCompat.getMainExecutor(this), object : ImageCapture.OnImageSavedCallback { override fun onError(e: ImageCaptureException) { Toast.makeText( baseContext, "Error al capturar imagen", Toast.LENGTH_SHORT).show() Log.e("CameraX",e.toString()) } override fun onImageSaved( output: ImageCapture.OutputFileResults

) {

correctamente!",

Toast.makeText( baseContext, "La imagen ${output.savedUri} se guardó

Toast.LENGTH_SHORT).show() Log.d("CameraX", output.savedUri.toString()) } } ) } }
