# ForegroundとBackgroundの検知

アプリのForegroundとBackgroundに入ったことを検知する仕組みは
MonoBehaviour.OnApplicationPause(bool)でお手軽に実現できる。

```

private void OnApplicationPause(bool pause)
{
    if(pause)
    {
        //Backgroundに入った時の処理
    }
    else
    {
        //Foregroundに復帰した時の処理
    }
}

```
