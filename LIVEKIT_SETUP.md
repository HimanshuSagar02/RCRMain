# LiveKit Integration Setup Guide

## Overview
LiveKit has been integrated into the RCR platform for live classes. When educators start a live class using the "Portal" platform, it automatically creates a LiveKit room for real-time video/audio streaming.

## Environment Variables

Add these to your `backend/.env` file:

```env
# LiveKit Configuration
LIVEKIT_API_KEY=APIgxiUFATz9e75
LIVEKIT_API_SECRET=your-livekit-api-secret-here
LIVEKIT_URL=wss://rcrtech-kla69owt.livekit.cloud
```

**Important:** You need to get your `LIVEKIT_API_SECRET` from your LiveKit Cloud dashboard:
1. Go to https://cloud.livekit.io/
2. Navigate to your project settings
3. Copy the API Secret
4. Add it to your `.env` file

## How It Works

### For Educators:
1. Create a live class and select "Portal" as the platform
2. When ready, click "Start & Join" - this:
   - Changes status to "live"
   - Creates a LiveKit room (room name: `liveclass-{liveClassId}`)
   - Opens the LiveKit player
3. Students can join once the class is live

### For Students:
1. View upcoming/live classes in their dashboard
2. Click "Join Live Class" when the class is live
3. Automatically connects to the LiveKit room
4. Can view educator's video/audio and participate in chat

## Features

- ✅ Real-time video/audio streaming
- ✅ Screen sharing (educators)
- ✅ Chat functionality
- ✅ Multiple participants support
- ✅ Camera/microphone controls
- ✅ Automatic room creation when class starts

## API Endpoints

- `GET /api/liveclass/:liveClassId/livekit-token` - Get LiveKit access token
- `PATCH /api/liveclass/:liveClassId/status` - Update class status (creates room when set to "live")

## Troubleshooting

1. **Connection Failed**: 
   - Check if `LIVEKIT_API_SECRET` is set correctly
   - Verify the LiveKit URL is correct
   - Ensure the class status is "live"

2. **No Video/Audio**:
   - Check browser permissions for camera/microphone
   - Verify LiveKit room was created (check database for `liveKitRoomName`)

3. **Token Generation Error**:
   - Verify API key and secret in `.env`
   - Check backend logs for detailed error messages

## Notes

- LiveKit rooms are automatically created when a class status changes to "live"
- Room names follow the pattern: `liveclass-{liveClassId}`
- Only portal-based classes use LiveKit (Zoom/Google Meet use external links)
- All participants can publish video/audio, but educators have full control

