## How to Implement Object Pooling?

First, we have to create a new script, let’s say BulletService which will be a generic singleton.

Like this -
```C#
public class BulletService : MonoSingletonGeneric<BulletService>
    {
        [SerializeField]
        private BulletController bulletPrefab;

        [SerializeField]
        private int poolSize;
        private List<BulletController> bulletPool;

        private void Start()
        {
            bulletPool = new List<BulletController>();
            for (int i = 0; i < poolSize; i++)
            {
                CreateBulletAndAddToPool();
            }
        }

        private void CreateBulletAndAddToPool()
        {
            BulletController createdBullet = CreateBullet();
            bulletPool.Add(createdBullet);
            createdBullet.gameObject.SetActive(false);
        }

        public BulletController GetBulletFromPool()
        {
            if (bulletPool.Count <= 0)
            {
                CreateBulletAndAddToPool();
            }
            return bulletPool.Remove();
        }

        private BulletController CreateBullet()
        {
            BulletController createdBullet = Instantiate(bulletPrefab);
            return createdBullet;
        }

        public void AddBulletToPool(BulletController bullet)
        {
            bulletPool.Add(bullet);
            bullet.gameObject.SetActive(false);
        }
    }
```
We have the BulletService script, which will give us the bullet whenever needed, but we have to call this bullet from somewhere right? Let’s see how we can do that 

Like this -
```C#
[RequireComponent(typeof(Rigidbody))]
    public class BulletController: MonoBehaviour
    {
        private Rigidbody rb;
        private Vector3 spawnPos;

        private void Awake()
        {
            rb = GetComponent<Rigidbody>();
        }

        public void Fire(Vector3 eulerAngle, float shootSpeed)
        {
            rb.velocity = Vector3.zero;
            rb.angularVelocity = Vector3.zero;
            transform.eulerAngles = eulerAngle;
            rb.AddForce(transform.forward * shootSpeed);
            spawnPos = transform.position;
        }

        private void OnCollisionEnter(Collision other)
        {
	    //if the bullet collides with an enemy tank or player tank, call damage function. 
            BulletService.Instance.AddBulletToPool(this);
        }
    }
```
We have the BulletController script, now we have to shoot the bullet from somewhere, right?

We can shoot the bullet from TankController script.

like this -
```C#
public class TankController: MonoBehaviour
{
        public void ShootBullet()
        {
            BulletController bullet = BulletService.Instance.GetBulletFromPool();
            bullet.gameObject.SetActive(true);
            bullet.transform.position = bulletSpawnPosition.position;
            bullet.Fire(turret.transform.eulerAngles, bulletSpeed);
        }
}
```
Now we are ready to use the object pooling for bullets in our battle tank game. Similarly, you can do this for enemy tanks also.
