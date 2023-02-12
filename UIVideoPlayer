using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.AddressableAssets;
using UnityEngine.ResourceManagement.AsyncOperations;
using MyBox;

namespace CastrimarisStudios.Video {

    /// <summary>
    /// A component that plays videos on UI elements.
    /// </summary>

    public class UIVideoPlayer : MonoBehaviour {

        #region ENUMS
        /// <summary>
        /// Where the video is located and how the component should load it.
        /// There are 4 types total:
        /// REFEFRENCE - the clip is assigned through the editor.
        /// STREAMINGASSETS - tries to dynamically load the clip from the StreamingAssets folder by the name specified in clipName.
        /// RESOURCES - tries to load the clip from the Resources internal folder by the name specified in clipName.
        /// ADDRESSABLE - tries to dynamically load the clip from Addressable assets by the name specified in clipName.
        /// </summary>
        public enum ClipType { REFERENCE, STREAMINGASSETS, RESOURCES, ADDRESSABLE }
        #endregion

        #region PRIVATE VARIABLES

        private const string TAG = "UIVideoPlayer";

        private Coroutine playbackCoroutine = null;
        private AsyncOperationHandle clipHandle;

        [Header("Parameters")]
        [Tooltip("The type of the clip. There are 4 types total:")]
        [SerializeField] private ClipType clipType = ClipType.REFERENCE;
        [ConditionalField(nameof(clipType), false, ClipType.REFERENCE)]
        [Tooltip("Reference to the clip. Needs to be assigned only if the clipType is REFERENCE.")]
        [SerializeField] private UnityEngine.Video.VideoClip clip = null;
        [ConditionalField(nameof(clipType), true, ClipType.REFERENCE)]
        [Tooltip("Name of the clip. Needs to be specified only if clipType isn't REFERENCE.")]
        [SerializeField] private string clipName = null;
        [Tooltip("Whether the video should loop or not.")]
        [SerializeField] private bool should_Loop = true;

        [Header("References")]
        [Tooltip("Reference to the videoplayer. If it's not assigned, the component creates one.")]
        [SerializeField] private UnityEngine.Video.VideoPlayer videoPlayer = null;
        [Tooltip("Reference to the RawImage to play the video on. If the reference is not set, tries to get a RawImage from this.")]
        [SerializeField] private UnityEngine.UI.RawImage target = null;
        #endregion

        #region PRIVATE_FUNCTIONS
        /// <summary>
        /// Coroutine to play the video when it's ready.
        /// </summary>
        private IEnumerator PlayVideo() {
            if (clipType == ClipType.ADDRESSABLE) {
                yield return clipHandle;
            }
            videoPlayer.Prepare();
            WaitForSeconds wfs = new WaitForSeconds(.5f);
            while (!videoPlayer.isPrepared) {
                yield return wfs;
            }
            target.GetComponent<UnityEngine.UI.RawImage>().texture = videoPlayer.texture;
            videoPlayer.Play();
        }

        /// <summary>
        /// Callback function to be called when the loading of the Clip from Addressables has completed.
        /// </summary>
        /// <param name="ClipHandle">The calling Handle</param>
        private void OnAsyncClipLoadCompleted(AsyncOperationHandle ClipHandle) {
            videoPlayer.clip = (UnityEngine.Video.VideoClip)clipHandle.Result;
        }

        #endregion

        #region PUBLIC_FUNCTIONS

        /// <summary>
        /// Sets the Clip to show on the UI element.
        /// </summary>
        /// <param name="Clip">The clip to set.</param>
        public void SetClip(UnityEngine.Video.VideoClip Clip) {
            if (clipType != ClipType.REFERENCE) {
                Debug.LogWarning($"-{TAG}- Tried to set the clip of the video, but ClipType is not set to Reference.");
            } else {
                clip = Clip;
            }
        }

        #endregion

        #region UNITY OVERRIDES

        private void Awake() {
            if (target == null) {
                target = GetComponent<UnityEngine.UI.RawImage>();
                if (target == null) {
                    Debug.LogError($"-{TAG}- Undefined target. Did you forget to assign this script to a RawImage?");
                    return;
                }
            }
            if (clip == null && clipName == null) {
                Debug.LogError($"-{TAG}- Undefined videoclip. Did you forget to assign a videoclip to this script?");
                return;
            }
            if (videoPlayer == null) {
                videoPlayer = this.gameObject.AddComponent<UnityEngine.Video.VideoPlayer>();
            }

            //Clip dynamic assignment
            switch (clipType) {
                case ClipType.REFERENCE:
                    videoPlayer.source = UnityEngine.Video.VideoSource.VideoClip;
                    videoPlayer.clip = clip;
                    break;
                case ClipType.RESOURCES:
                    videoPlayer.source = UnityEngine.Video.VideoSource.VideoClip;
                    videoPlayer.clip = Resources.Load<UnityEngine.Video.VideoClip>(clipName);
                    break;
                case ClipType.STREAMINGASSETS:
                    videoPlayer.source = UnityEngine.Video.VideoSource.Url;
                    videoPlayer.url = System.IO.Path.Combine(Application.streamingAssetsPath, clipName+".mp4");
                    break;
                case ClipType.ADDRESSABLE:
                    Caching.compressionEnabled = false; //necessary because, otherwise, Android tries to compress videos first, taking a while to load.
                    videoPlayer.source = UnityEngine.Video.VideoSource.VideoClip;
                    clipHandle = Addressables.LoadAssetAsync<UnityEngine.Video.VideoClip>(clipName);
                    clipHandle.Completed += OnAsyncClipLoadCompleted;
                    break;
                default:
                    Debug.LogError($"-{TAG}- Unassigned clipType.");
                    break;
            }

            videoPlayer.renderMode = UnityEngine.Video.VideoRenderMode.APIOnly;
            videoPlayer.isLooping = should_Loop;
            videoPlayer.SetDirectAudioVolume(0, 0.0f);
        }

        private void OnEnable() {
            //Play the video whenever the button is enabled
            playbackCoroutine = StartCoroutine(PlayVideo());
        }

        private void OnDisable() {
            //Stop the video whenever the button is disabled
            StopCoroutine(playbackCoroutine);
        }

        #endregion

    }

}
