Google and Meta Authentication by Priyanshi Singh- DS, GGITS'28

from django.conf import settings
from django.conf.urls.static import static  # Import the static function
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),  # Admin panel
    path('', include('home.urls')),  # Include URLs from the "home" app
]

# Serve media files in development
if settings.DEBUG:
    urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)






